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
to be solved at the command line (bash). The exam is currently provisioned on Ubuntu
18.04. The questions are weighted differently.  You have 2 hrs to complete the
CKA. The exam is currently based on Kubernetes v1.21. There are six clusters
that comprise the CKA exam environment.  Each with one main node, a varying
number of worker nodes, and different CNI's.  The 'context' you need to operate
in will be specified at the start of each question. You are allowed to open one
additional browser tab to access documentation, manifests, etc., from:
  - https://kubernetes.io/docs
  - https://github.com/kubernetes/
  - https://kubernetes.io/blog

## Practice environments
- katacoda.com - a browser-based interactive playground
- Minikube - local kubernetes cluster installation
- Public cloud providers - from scratch using VMs on either AWS, Azure, GCP, etc.
- killer.sh - simulator for CKS, CKA and CKAD.
- kubewiz.com - similar offering to killer.sh

## Be familiar with:
- linux - (ps, systemd, journald, iptables)
- a text editor (nano, vim) - to the extent that you are comfortable editing yaml
- YAML - and to a lesser extent JSON
- docker

## Tips 

### Bookmarks
- While you can search in kubernetes.io, having bookmarked relevant domain content will
help you find examples more quickly during the exam. Useful bookmarks:  
    - kubectl cheat sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
    - kubectl reference docs: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
- Better still, familiarise yourself with `kubectl explain` (e.g. `kubectl explain pod.spec.containers --recursive` to see overall yaml structure).

### VIM
#### as a suggestion:
Create a vim configuration file: `vim ~/.vimrc` and once you have saved the
following, don't forget to source it: `source ~/.vimrc`
```
"Comment: setting to enable vim to deal with yaml indentation requirements
autocmd FileType yaml setlocal ts=2 sts=2 sw=2 et
"Comment: turn on line numbering
:set number
```
### BASH
#### as a minimum suggestion:
- To add autocomplete permanently to your bash shell: 
  - `echo "source <(kubectl completion bash)" >> ~/.bashrc`
- To add a shorthand alias for 'kubectl' permanently to your bash shell, that also works with autocomplete:
  - `echo "alias k=kubectl" >> ~/.bashrc`
  - `echo "complete -F __start_kubectl k" >> ~/.bashrc"`
- Other useful alias:
  - `echo "alias kcc='kubectl config current-context'" >> ~/.bashrc`
- Note: the exam will require you to ssh into different nodes. 
- Remember to:
  - `source .bashrc`
#### as an optional suggestion:
    - `echo "export dry='--dry-run=client -o yaml'" >> ~/.profile`
- To generate a dry run yaml output, you could for example run: `k run test-pod-1 --image=nginx $dry`
    - `echo "export fd='--force --grace-period=0'" >> ~/.profile`
- To force delete a pod (or other resource), you could for example run `k delete
  test-pod-2 $fd`

### TMUX (tmux is not required, but is available should you require it)
- Create a tmux configuration file, e.g. `vim ~/.tmux.conf`
  - `set-environment -g PATH $PATH # persists the bash env variables set above in tmux session`

## Repos:
- A series of practice exercises that can be performed in a practice environment 
