# K8s

Setup a 3 node Kubernetes cluster.

## Usage

Secrets files to be created:

1. Create _~/.ansible_secret/vault_pass_insecure_ (relative to home folder)
1. Create _~/.ansible_secret/vault_pass_sudo_ to store sudo password in production
1. Create _./ansible/vault_pass_prod_ to retain sudo password in production (in the project folder)

If you want different secrets edit _ansible.cfg_ and change `vault_identity_list` to match yout setup. Also modify Vagrantfile as needed.The file _vault_pass_insecure_ is used for development and shared between projects. It may be changed to a project specific secret located in the ansible project directory. The secret file _vault_pass_sudo_ is used to store the variabl `ansible_become_pass` with the sudo password to mitigate password typing.
