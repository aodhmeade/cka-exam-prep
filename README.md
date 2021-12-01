# CKA-exam-notes 
Some notes & other material to help prepare for the Certified Kubernetes
Administrator (CKA) exam (v1.21).

## notes for each domain 
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
18.04. The questions are weighted differently.  You have 2 hrs to complete the
CKA. The exam is currently based on Kubernetes v1.21. There are six clusters
that comprise the CKA exam environment.  Each with one main node, a varying
number of worker nodes, and different CNI's.  The 'context' you need to operate
in will be specified at the start of each question. You are allowed to open one
additional browser tab to access documentation, manifests, etc., from:
  - https://kubernetes.io/docs
  - https://github.com/kubernetes/
  - https://kubernetes.io/blog

## some means of practice (no affiliation)
- katacoda.com - a browser-based interactive terminal playground.
- minikube - install minikube for a local kubernetes cluster setup (certain hw
  requirements, such as 2cpus, etc.)
- public cloud providers - from scratch using virtual machines on either AWS, Azure, GCP, etc.
- killer.sh - a test simulator for the CKS, CKA and CKAD exams.

## useful to be familiar with:
- linux - systemd, journald, iptables, grep, ps, curl ...
- text editor (vim) - to the extent that you are comfortable editing yaml (nano, gedit maybe
  sufficient, but you may have to install).
- YAML - be aware of syntax (and JSON syntax to a lesser extent)
- docker - basic knowledge (know how to check status, images, logs, etc.).

## tips 

### create some kubernetes.io bookmarks
#### as a suggestion:

- While you can utilise the search functionality in kubernetes.io during the
  exam, having bookmarked relevant domain content will help you find examples
  quickly during the exam. Useful bookmarks:  
    - kubectl cheat sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
    - kubectl reference docs: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
- Ultimately, familiarising yourself with 'kubectl explain' should prove more
  beneficial in the long run. (e.g. `kubectl explain pod.spec.containers
  --recursive` to see overall yaml structure). 

### vim
#### as a suggestion:
Create a vim configuration file: `vim ~/.vimrc` and once you have saved the
following to help deal with yaml indentation and optionally to add line
numbering, don't forget to read the configuration into the current shell
environment: `source ~/.vimrc`
```
"Comment: setting to enable vim to deal with yaml indentation requirements
autocmd FileType yaml setlocal ts=2 sts=2 sw=2 et
"Comment: turn on line numbering
:set number
```
### bash
#### as a suggestion:
- To add autocomplete permanently to your bash shell: 
  - `echo "source <(kubectl completion bash)" >> ~/.bashrc`
- To add a shorthand alias for 'kubectl' permanently to your bash shell, that also works with autocomplete:
  - `echo "alias k=kubectl" >> ~/.bashrc`
  - `echo "complete -F __start_kubectl k" >> ~/.bashrc"`
- Another useful alias when you want to double check that you are operating in
  the correct context:
  - `echo "alias kcc='kubectl config current-context'" >> ~/.bashrc`
- Note: given that the exam will require you to ssh into different nodes, you
  may not get the benefit of spending time setting up a large number of aliases.
- Remember to read in the configuration file to the current shell environment:
  - `source .bashrc`

#### as an optional suggestion:
- To create an environment variable to generate a dry-run yaml output:
`echo "export dry='--dry-run=client -o yaml'" >> ~/.profile`
- You could for example then run: `k run test-pod-1 --image=nginx $dry`

- To force delete a pod (or other resource), you could for export
`echo "export fd='--force --grace-period=0'" >> ~/.profile`
- You could for example then run: `k delete test-pod-2 $fd`

### tmux (tmux is not required, but is available should you require it)
#### as a suggestion:
- Create a tmux configuration file, e.g. `vim ~/.tmux.conf`
  - `set-environment -g PATH $PATH # persists the bash env variables set above
    should you create a tmux session`

## useful repos:
credit @dgkanatsios and co. for a set of practice exercises for CKAD which are a
helpful crossover for the CKA exam.
[https://github.com/dgkanatsios/CKAD-exercises] 
