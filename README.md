# Deploy a High Availability Kubernetes Storidge Cluster with Terraform on AWS

![Logo](https://i.imgur.com/FfIj2NA.png) ![Kubernetes Logo](https://raw.githubusercontent.com/kubernetes-sigs/kubespray/master/docs/img/kubernetes-logo.png) 

Check out https://github.com/kubernetes-sigs/kubespray for kubespray documentation

This project will create:
* A High Availability Kubernetes cluster with Storidge persistent storage
* VPC with Public and Private Subnet
* Bastion Hosts and NAT Gateways in the Public Subnet
* A dynamic number of masters, etcd, and worker nodes in the Private Subnet
* AWS ELB in the Public Subnet for accessing the Kubernetes API from the internet

## Quick Start

For requirements, check https://github.com/kubernetes-sigs/kubespray#requirements

In root folder:
```ShellSession
# Install dependencies from ``requirements.txt``
sudo pip3 install -r requirements.txt
```
**How to Use:**
Go into the right directory: 
```cd /contrib/terraform/aws/```:
- Export the variables for your AWS credentials:

```
export TF_VAR_AWS_ACCESS_KEY_ID="www"
export TF_VAR_AWS_SECRET_ACCESS_KEY ="xxx"
export TF_VAR_AWS_SSH_KEY_NAME="yyy"
export TF_VAR_AWS_DEFAULT_REGION="zzz"
```

 or edit `credentials.tfvars`

- Update `contrib/terraform/aws/terraform.tfvars` with your data. By default, the Terraform scripts use Ubuntu 18.04 LTS (Bionic) as base image. This image works with Storidge guaranteed.
- Create an AWS EC2 SSH Key
- Run `terraform init`
- Run with `terraform apply --var-file="credentials.tfvars"` or `terraform apply` depending if you exported your AWS credentials

Example:
```commandline
terraform apply -var-file=credentials.tfvars
```

- Terraform automatically creates an Ansible Inventory file called `hosts` with the created infrastructure in the directory `inventory`

- Ansible will automatically generate an ssh config file for your bastion hosts. To connect to hosts with ssh using bastion host use generated ssh-bastion.conf.
  Ansible automatically detects bastion and changes ssh_args  
```commandline
ssh -F roles/bastion-ssh-config/templates/ssh-bastion.conf user@$ip
```

NOTE: This likely will not work, so edit `.ssh/config` instead:

```
Host bastion
  Hostname (bastion host ip here)
  StrictHostKeyChecking no
  ControlMaster auto
  ControlPath ~/.ssh/ansible-%r@%h:%p
  ControlPersist 5m

Host (bastion host ip here)
  ProxyCommand ssh -F /dev/null -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -W %h:%p ubuntu@(bastion host ip here) -i ~/.ssh/kubespray.pem

Host (node ip here)
  ProxyCommand ssh -F /dev/null -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -W %h:%p ubuntu@(node ip here) -i ~/.ssh/kubespray.pem
  ```
  
- Add last 2 lines for as many nodes as necesarry.

- Once the infrastructure is created, you can run the kubespray playbooks and supply inventory/hosts with the `-i` flag.

Run Ansible in root folder:
```commandline
ansible-playbook -i ./inventory/hosts ./cluster.yml -e ansible_user=ubuntu -b --become-user=root
-e ansible_ssh_private_key_file=~/.ssh/myawsprivatekey.pem --flush-cache
```

