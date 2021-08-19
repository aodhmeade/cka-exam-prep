# cka-exam-prep
Some notes & other relevant material to help me prepare for the Certified Kubernetes Administrator (CKA) exam.  

## Exam domains & weights 
| Exam Domain                                       | Weight |
|--------------------------------------------------------|:------:|
| [Cluster Architecture, Installation & Configuration](cluster-architecture-installation-and-configuration.md) |   25%  |
| [Workloads & Scheduling](workloads-and-scheduling.md)                             |   15%  |
| [Services & Networking](services-and-networking.md)                              |   20%  |
| [Storage](storage.md)                                            |   10%  |
| [Troubleshooting](troubleshooting.md)                                    |   30%  |

# Kubernetes documentation
- [https://kubernetes.io/]

## Exam preparation
- Official guidance: exam details, registration, etc., can be found at
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
- Bookmarks
- Aliases

## Day of exam

## Books, repos, blogs I've found useful ...

