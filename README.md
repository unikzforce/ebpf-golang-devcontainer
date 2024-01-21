# go + ebpf development using devcontainer + docker (even works in apple silicon)


this is a starter project in which you can start developing `ebpf` without any hassle.
it creates an ebpf ready environment using `devcontainer`+`docker`.

it even works in `apple silicon` (m1/m2/m3...) macOS.

(actually I have only tested it in mac, will test it in other devices as well if I get the chance)

if you don't know what a `devcontainer` is, it is a shareable, reusable & reproducible 
development environment, which is supported by docker and won't pollute your system or VM.
so you don't need to create an ebpf development environment yourself,
and most importantly you can throw it away anytime.


## Prerequisites

- vscode or goland or neovim (any IDE which supports remote container development)
  - for neovim I haven't tested yet, but the [neo-vim-remote-containers](https://github.com/jamestthompson3/nvim-remote-containers) can help 
- docker ( personally I'm enjoying [OrbStack](https://orbstack.dev/) for macOS, but I guess any docker flavour would be ok)


## how to import project in JetBrains GoLand

- open the project with goland
- choose the container icon when opening up `devcontainer.json` and click on `Create Dev Container and Mound Sources...`

![JetBrains GoLand](./img/1_run_devcontainer_jetbrains.png?raw=true)

- if it wants to access your confidential information feel free to `deny` it, currently I'm not sure why it's asking it and didn't want to share my information with it, so I denied it, and it worked flawlessly.  

![JetBrains GoLand](./img/2_run_devcontainer_jetbrains.png?raw=true)

## how to import project in VSCode

- I suppose you already have installed [Remote Development Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack) for vscode, I guess it should be installed by default
- open the project with vscode
- click on `Reopen in Container`

![JetBrains GoLand](./img/1_run_devcontainer_vscode.png?raw=true)

## how to run the example ebpf program?

when the container env has been loaded and get ready:
  - open an extra terminal in that container in your ide or directly create a terminal into that container outside ide. if you open up the terminal within the ide it should automatically navigate to project source folder in container itself.
  - run `go generate ./ebpf` it will generate some go files from `./ebpf/counter.c` bpf file using `bpf2go` tool. you can use these files in your `./cmd/main.go`. it would greatly simplify the userspace/kernel space communication. **the generated files are already committed, so you won't feel any difference after executing this command**.  
  - run `go run ./cmd/main.go` in jetbrains goland, in vscode you need to run `sudo go run ./cmd/main.go` ( currently I'm not sure why, but I'll try to fix it in the future.)


## a little technical explanation:

- usually ebpf programs has two legs, one leg in the userspace (`./cmd/main.go`) and the other leg in kernel space (`./ebpf/counter.c`)
- we write the ebpf code (kernel space code) in `c` (or something similar to ) and put it into a file, in our case `./ebpf/counter.c`.
- we need to compile the kernel space code (`./ebpf/counter.c`) and then load it into the kernel, upon loading the kernel would first verify the compiled object and if it agrees with it then load it into kernel
- we can do this procedure manually and step by step, but it would be a tedious task
- here comes the [cilium ebpf project](https://github.com/cilium/ebpf), which helps us to do the compilation and loading the kernel space to the program an easy task
- using `go generate ./ebpf` + `./ebpf/gen.go` we ask the `bpf2go` to compile the `./ebpf/counter.c` and also generate some extra golang codes
- then in `./cmd/main.go` userspace code we use those generated go codes and actually try to load the compiled version of `./ebpf/counter.c` into the kernel space.
- after the ebpf program get loaded in the kernel, we can maintain a communication channel between the kernel space and userspace, so in `./cmd/main.go` you can do some extra work as well
  - like fetching statistics that has been sent by the kernel space program
  - ....
- in ebpf there are some data structures that can hold data, which is called `bpf maps`, the communication between userspace and kernel space can be achieved through these `bpf maps`


## Thanks to
- [cilium ebpf project](https://github.com/cilium/ebpf) for creating `bpf2go`, check out their website [ebpf-go.dev](https://ebpf-go.dev/) for more learning material on ebpf and go.
- https://github.com/markpash for introducing `bpf2go` to me