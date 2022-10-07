# AZURE-VIRTUAL-MACHINE

This repository consists of Terraform templates to create Azure Virtual Machine object.

## Usage

- Clone this repo with: `git clone --recurse-submodules https://github.com/cklewar/azure-virtual-machine`
- Enter repository directory with: `cd azure-virtual-machine`
- Obtain F5XC API certificate file from Console and save it to `cert` directory
- Pick and choose from below examples and add mandatory input data and copy data into file `main.tf.example`.
- Rename file __main.tf.example__ to __main.tf__ with: `rename main.tf.example main.tf`
- Initialize with: `terraform init`
- Apply with: `terraform apply -auto-approve` or destroy with: `terraform destroy -auto-approve`

## Create Azure Virtual Network


```hcl

```