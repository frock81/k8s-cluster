# K8s

Setup a 3 node Kubernetes cluster.

## Preparation

Clone and cd to the project directory

    $ git clone https://github.com/frock81/k8s-cluster.git
    $ cd k8s-cluster

## Secrets

Secrets files to be created:

1. _~/.ansible_secret/vault_pass_insecure_ (relative to the home folder)
1. _~/.ansible_secret/vault_pass_sudo_ (to store sudo password for production)
1. _./ansible/vault_pass_prod_ (relative to the project folder to store sudo password for production)

Substitute the passwords accordingly.

```
    $ mkdir ~/.ansible_secret
    $ echo 'DEVELOPMENTPASSWORD' > ~/.ansible_secret/vault_pass_insecure
    $ echo 'RANDOMPASSWORD' > ~/.ansible_secret/vault_pass_sudo
    $ echo 'PRODUCTIONPASSWORD' > ansible/.vault_pass_prod
```

If you want different secrets edit _ansible.cfg_ and change `vault_identity_list` to match yout setup. Also modify Vagrantfile as needed (`~/.ansible_secret` synced folder).

The file _vault_pass_insecure_ is used for development and shared between projects, so, as the name suggests, it is not secure to use it in production. It may be changed to a project specific secret located in the ansible project directory.

The secret file _vault_pass_sudo_ is used to store the variable `ansible_become_pass` with the sudo password to ease password typing. I used to put my development files in Dropbox/Google drive folder. In order to not store my sudo password I vaulted it. As such I put the key away from the vault (in _~/.ansible_secret_). The same way as the _vault_pass_insecure_, create a file with a random password in it (it does not need to be remembered as if it is deleted of forgotten another one can be generated). This step is optional, but if you want to use it run:

    $ cd ansible
    $ mkdir -p group_vars/all
    $ ansible-vault create --encrypt-vault-id=sudo group_vars/all/secret-local.yml

The file _secret-local.yml_, **should not** be put in version controle, but if it is by mistake, at least it is encrypted and your peers from the project won't have the password (different from _vault_pass_insecure_ and probably _.vault_pass_prod_).

## Running

    $ vagrant up

## References

- https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/
- https://github.com/geerlingguy/ansible-role-docker
- https://github.com/geerlingguy/ansible-role-pip
- https://github.com/geerlingguy/ansible-role-kubernetes
