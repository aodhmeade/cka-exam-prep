# cka-exam-prep
Some notes & other relevant material to help prepare for the Certified Kubernetes Administrator (CKA) exam.  

## exam domains & weights 
| Exam Domain                                       | Weight |
|--------------------------------------------------------|:------:|
| [Cluster Architecture, Installation & Configuration](cluster-architecture-installation-and-configuration.md) |   25%  |
| [Workloads & Scheduling](workloads-and-scheduling.md)                             |   15%  |
| [Services & Networking](services-and-networking.md)                              |   20%  |
| [Storage](storage.md)                                            |   10%  |
| [Troubleshooting](troubleshooting.md)                                    |   30%  |

# Kubernetes documentation
[https://kubernetes.io/]

## exam preparation
Official guidance: exam details, registration, etc., can be found at
  [CNCF](https://www.cncf.io/certification/cka/) and at the [Linux
  Foundation](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/),
  as this is a joint effort between both. **Read:** candidate handbook, exam
  tips, curriculum overview, faqs. **Watch:** a demo of the exam environment.

## Note
The exam is proctored remotely. It consists of 15-20 performance-based problems
to be solved at the command line. The exam is currently provisioned on Ubuntu
18.04. Likely standard shells available: sh, bash. The questions are weighted
differently.  You have 2 hrs to complete the CKA. The exam is currently based on
Kubernetes v1.21. There are six clusters that comprise the CKA exam environment.
Each with one main node, a varying number of worker nodes, and different CNI's.
The 'context' you need to operate in will be specified at the start of each
question. You are allowed to open one additional browser tab to access
documentation from:
  - https://kubernetes.io/docs
  - https://github.com/kubernetes/
  - https://kubernetes.io/blog

## Practice environments
- Katacoda - browser-based interactive playground
- Minikube - local installation
- From scratch using VMs on either GCP, AWS, Azure, etc.
- killer.sh (check this out ...)
- https://kubewiz.com/exams  ... check this out

## Be familiar with:
- (systemd ... running services)
- a text editor (nano, vim)
- YAML/JSON
- Docker
- tmux (optional)

## Tips 

### Bookmarks

### VIM
Create a vim configuration file: `vim ~/.vimrc` and once you have saved the
following, don't forget to source it: `source ~/.vimrc`
```
inoremap jj <ESC>
:set relative number
:set number
autocmd FileType yaml setlocal ts=2 sts=2 sw=2 et
```
### BASH
- To add autocomplete permanently to your bash shell: 
  - `echo "source <(kubectl completion bash)" >> ~/.bashrc`
- To add a shorthand alias for 'kubectl' permanently to your bash shell, that also works with autocomplete:
  - `echo "alias k=kubectl" >> ~/.bashrc`
  - `echo "complete -F __start_kubectl k" >> ~/.bashrc"`
- Other useful alias:
  - `echo "alias kcc='kubectl config current-context'" >> ~/.bashrc`
- Remember to:
  - `source .bashrc`
- `echo "export dry='--dry-run=client -o yaml'" >> ~/.profile`
- `echo "export fd='--force --grace-period=0'" >> ~/.profile`

### TMUX (tmux is not required, but useful to have the option). 
- Create a tmux configuration file, e.g. `vim ~/.tmux.conf`
  - `set -g base-index 1 # changes the default window numbering to start from 1 rather than 0`
  - `set-environment -g PATH $PATH # persists the bash env variables set above in tmux session`

## Repos:

