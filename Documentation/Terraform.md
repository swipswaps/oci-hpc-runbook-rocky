# <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/rockylogo.png" height="40"> Runbook

# Deployment through Terraform Script

**Table of Contents**
- [Deployment through Terraform Script](#deployment-through-terraform-script)
  - [Terraform Installation](#terraform-installation)
  - [Using terraform](#using-terraform)
    - [Configure](#configure)
    - [Run](#run)
    - [Destroy](#destroy)

## Terraform Installation

Download the binaries on the [terraform website](https://www.terraform.io/) and unzip the package. Depending on your Linux distribution, it should be similar to this:

```
tf_install_dir=~/tf_install_dir
cd $tf_install_dir
wget https://releases.hashicorp.com/terraform/0.12.0/terraform_0.12.0_linux_amd64.zip
unzip terraform_0.12.0_linux_amd64.zip
echo export PATH="\$PATH:$tf_install_dir" >> ~/.bashrc
source ~/.bashrc
```

To check that the installation was done correctly: `terraform -version` should return the version. 

## Using terraform
### Configure
Download the [zip](https://github.com/oci-hpc/oci-hpc-runbook-rocky/raw/master/Resources/tf_rocky.zip) file and unzip the content.

Edit the file terraform.tfvars for your settings, info can be found [on the terraform website](https://www.terraform.io/docs/providers/oci/index.html#authentication)

* Tenancy_ocid
* User_ocid
* Compartment_ocid
* Private_key_path
* Fingerprint
* Region

**Note1: For Compartment_ocid: To find your compartment ocid, go to the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/menu.png" height="20"> and select Identity, then Compartments. Find the compartment and copy the ocid.**

<img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/compartment_OCID.png" height="150">

**Note2: The private_key_path and fingerprint are not related to the ssh key to access the instance. You can create using those [instructions](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/apisigningkey.htm).**


In the variable.tf file, you can change the availability domain, the shapes of the nodes, the number of visualization nodes, the VNC settings, Rocky installer URL,... The different variables are explained in the [Ressource Manager section](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/Documentation/ResourceManager.md#select-variables)

### Run
```
cd <folder>
terraform init
terraform plan
terraform apply
```

**If you wish to add or remove nodes after the setup has happened, just modify the variable in the variable.tf file and rerun the `terraform apply` command**

### Destroy
```
cd <folder>
terraform destroy
```

## Access your cluster

Once you have created your cluster, if you gave a valid URL for the Rocky installation, no other action will be needed except [running your jobs](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/README.md#running-the-application).

Public IP addresses are written at the end of the run. 

The key to log on to your cluster has been created in your main directory as key.pem

```
ssh -i /home/user/key.pem opc@ipaddress
```

Access to the visualization instances can be done through a SSH tunnel:

```
ssh -i /home/user/key.pem -x -L 5902:127.0.0.1:5900 opc@ipaddress
```

And then connect to a VNC viewer with localhost:2.

[More information](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/Documentation/ManualDeployment.md#accessing-a-vnc) about using a VNC session. 
