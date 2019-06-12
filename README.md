# <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/rockylogo.png" height="40"> Runbook

## Introduction
This Runbook will take you through the process of deploying a GPU machine on Oracle Cloud Infrastructure, installing Rocky DEM, configuring the license, and then running a model.

Rocky DEM simulates particles and can be coupled with Computational Fluid Dynamic or Finite Element Method. It adds another layer of complexity which results in increased simulation time. To help speed up the run, Rocky DEM provides the option to parallelize to a high number of CPU. For even more speed, it can unleash the full power of one or multiple GPUs. 

Running Rocky on Oracle Cloud Infrastructure, there is no special setup needed and no driver headache. Import your model, choose the number of CPUs or GPUs and off you go. This removes the wait time for resources that you may have on your on-premise cluster. It avoids having people battling for high-end GPUs at peak times and having them idle for the rest of the week.

![](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/3184v0.gif "Example DEM simulation in Rocky") 
 
## Architecture
The architecture for this runbook is simple, a single GPU machine running inside of an OCI VCN. If you are using a GPU machine, you will need to attach a block storage or a file system. If an HPC shape is chosen, you will have a local NVMe drive attached to your instance. 

![](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/HPC_arch_draft.png "GPU Architecture for Running Rocky in OCI")
## Deployment
There are multiple options available to get started with Rocky on OCI. The next 2 sections will show how to do it from the console in a webbrowser and using a Terraform script. Scripts are especially usefull with more complex architecture.    
### Console
#### Log In
You can start by logging in the Oracle Cloud console. If this is the first time, instructions to do so are available [here](https://docs.cloud.oracle.com/iaas/Content/GSG/Tasks/signingin.htm).
Select the region in which you wish to create your instance. Click on the current region in the top right dropdown list to select another one. 

<img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/Region.png" height="50">

#### Virtual Cloud Network
Before creating an instance, we need to configure a Virtual Cloud Network. Select the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/menu.png" height="20"> on the top left, then select Networking and Virtual Cloud Networks. <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/create_vcn.png" height="25">

On the next page, select the following: 
* Name of your VCN
* Compartment of your VCN
* Choose "CREATE VIRTUAL CLOUD NETWORK PLUS RELATED RESOURCES"

Scroll all the way down and <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/create_vcn.png" height="25">

#### Compute Instance
Create a new instance by selecting the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/menu.png" height="20"> on the top left, then select Compute and Instances. 

<img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/Instances.png" height="300">

On the next page, select <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/create_instance.png" height="25">

On the next page, select the following:
* Name of your instance
* Availibility Domain: Each region has multiple availability domain. Some instance shapes are only available in certain AD.
* Change the image source to CentOS 7.
* Instance Type: Select Bare metal
* Instance Shape: 
  * For 8 V100 GPU, select BM.GPU3.8
  * For 2 P100 GPU, select BM.GPU2.2
  * For CPUs, select BM.HPC2.36
  * Other shapes are available as well, [click for more information](https://cloud.oracle.com/compute/bare-metal/features).
* SSH key: Attach your public key file. For more information, click here.
* Virtual Cloud Network: Select the network that you have previsouly created.

Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/create.png" height="25">

After a few minutes, the instances will turn green meaning it is up and running. You can now SSH into it. After clicking on the name of the instance, you will find the public IP. You can now connect using `ssh opc@xx.xx.xx.xx` from the machine using the key that was provided during the creation. 

**If you are using Oracle Linux 7.6 on a machine with GPU's, during instance definition, selet the advanced options at the bottom, select image and make sure to take one that contains GPU in its name. That will remove the need to install any driver**

#### Block Storage

**If you are running an HPC shape to run Rocky on CPUs, you can skip this part as you already have local storage on your machine. However, you will still need to mount it**
Create a new Block Volume by selecting the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/menu.png" height="20"> on the top left, then select Block Storage and Block Volumes.

Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/create_bv.png" height="25">

On the next page, select the following: 
* Name
* Compartment
* Size (in GB)
* Availability Domain: Make sure to select the same as your Compute Instance. 

Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/create_bv.png" height="25">

Select the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/menu.png" height="20"> on the top left, then select Compute and Instances.

Click on the instance to which the drive will be attached.

On the lower left, in the Ressources menu, click on "Attached Block Volumes"

<img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/ressources.png" height="200">

Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/attach_BV.png" height="25">

All the default setting will work fine. Select the Block Volume that was just created and select /dev/oracleoci/oraclevdb as device path. 
Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/attach.png" height="25">
**Note: If you do not see the Block Volume, it may be because you did not place it in the same AD as your running instance**

Once it is attached, hit the 3 dots at the far right of the Block Volume description and select "iSCSi Commands and Information" 

<img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/ISCSi.png" height="150">

Copy the command to attach it to the instance. 

<img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/iscsi_commands.png" height="200">

Those commands will be used to mount the Block Volume to the instance. Log in to the machine `ssh opc@xx.xx.xx.xx` as seen at the end of the previous section. 

#### Mounting a drive

If you have local NVMe storage or if you have attached a block storage as seen in the previous section. You will need to mount it to your running instance to be able to use it. 

After logging in using ssh, run the command `lsblk`. 
The drive should be listed with the NAME on the left (Probably sdb for block and nvme0n1 for local storage, refered to as DNAME in the next commands)

Format the drive:
```
sudo mkfs -t ext4 /dev/DNAME
```

Create a directory and mount the drive to it.
```
sudo mkdir /mnt/disk1
sudo mount /dev/DNAME /mnt/disk1
sudo chmod 777 /mnt/disk1
```

### Terraform Script
#### Terraform Installation

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

#### Using terraform
##### Configure
Download the zip file for your needs and unzip the content:
* [GPU with Storage](https://github.com/oci-hpc/oci-hpc-runbook-rocky/raw/master/terraform_templates/tf_rocky_gpu.tar)
* [HPC Shape](https://github.com/oci-hpc/oci-hpc-runbook-rocky/raw/master/terraform_templates/tf_rocky_hpc.tar)

Edit the file terraform.tfvars for your settings, info can be found [on the terraform website](https://www.terraform.io/docs/providers/oci/index.html#authentication)

* Tenancy_ocid
* User_ocid
* Compartment_ocid
* Private_key_path
* Fingerprint
* SSH_private_key_path
* SSH_public_key
* Region

**Note1: For Compartment_ocid: To find your compartment ocid, go to the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/menu.png" height="20"> and select Identity, then Compartments. Find the compartment and copy the ocid.**

<img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/compartment_OCID.png" height="150">

**Note2: The private_key_path and fingerprint are not related to the ssh key to access the instance. You can create using those [instructions](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/apisigningkey.htm). The SSH public and private keys can be generated like [this](https://docs.cloud.oracle.com/iaas/Content/GSG/Tasks/creatingkeys.htm)**


In the variable.tf file, change the availability domain, the block storage size and the shape of the machine. 

##### Run
```
cd <folder>
terraform init
terraform plan
terraform apply
```
##### Destroy
```
cd <folder>
terraform destroy
```

## Installation
This guide will show the different steps for the CentOS 7 image available on Oracle Cloud Infrastructure. There is no need to configure NVIDIA GPUs if you have selected an Oracle LINUX version for GPU. 

### Configuring NVIDIA GPUs
If you are running Rocky on CPUs this part is not needed, you can skip directly to the part "Installing Rocky DEM".
By default, the nouveau open-source drivers are active. In order to install the Nvidia drivers, they need to be stopped. You can run the following script and reboot your machine. 
```
sudo yum -y install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
sudo sed -i 's/GRUB_CMDLINE_LINUX="/GRUB_CMDLINE_LINUX="modprobe.blacklist=nouveau /g' /etc/default/grub
sudo grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
sudo reboot
```
Once your machine has rebooted, you can run the command `lsmod | grep nouveau` and you should not see any results. 
```
sudo yum -y install kernel-devel-$(uname -r) kernel-headers-$(uname -r) gcc make redhat-lsb-core
wget https://developer.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.105_418.39_linux.run
sudo chmod +x cuda_10.1.105_418.39_linux.run
sudo ./cuda_10.1.105_418.39_linux.run --silent
```
### Installing Rocky DEM
There are a couple of library that need to be added to the CentOS image. 
```sudo yum -y install mesa-libGLU-devel mesa-libGL-devel```

You can download the Rocky installer using wget from the ESSS website or other place. In our architecture, we have mounted a file system in the /mnt directory.

```
rockyHomeDir=/mnt/disk1
sudo mkdir $rockyHomeDir/Rocky
sudo chmod 777 $rockyHomeDir/Rocky
sudo tar -jxvf rocky4-bin-4.2.0-linux64.tbz2 -C $rockyHomeDir/Rocky/
```

### (Optional) Set up a VNC
By default, the only access to the CentOS machine is through SSH in a console mode. If you want to see the Rocky interface, you will need to set up a VNC connection. The following script will work for the default user opc. The password for the vnc session is set as "password" but you can obvisouly edit that. 
```
sudo yum -y groupinstall "Server with GUI"
sudo yum -y install tigervnc-server mesa-libGL
sudo systemctl set-default graphical.target
sudo cp /usr/lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:0.service
sudo sed -i 's/<USER>/opc/g' /etc/systemd/system/vncserver@:0.service
sudo mkdir /home/opc/.vnc/
sudo chown opc:opc /home/opc/.vnc
echo "password" | vncpasswd -f > /home/opc/.vnc/passwd
chown opc:opc /home/opc/.vnc/passwd
chmod 600 /home/opc/.vnc/passwd
sudo firewall-offline-cmd --zone=public --add-port=5900-5901/tcp
sudo systemctl restart firewalld
sudo systemctl daemon-reload
sudo systemctl start vncserver@:0.service
sudo systemctl enable vncserver@:0.service
```
In the console, we need to allow access to the different port to allow access.

In Networking/Virtual Cloud Networks, select your VCN and the subnet in which your machine is located and Select the security list associated with it. 

Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/addIngress.png" height="20"> 
* CIDR : 0.0.0.0/0
* IP PROTOCOL: TCP
* Source Port Range: All
* Destination Port Range:5900-5901

After logging in for the first time, go to application/System Tools/Settings and in Power, set the timer for Blank Screen in Power saving to Never.

<img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/CentOSSeetings.jpg" height="150"> 

## Licensing

If you have enabled the VNC, you can start Rocky (`$rockyHomeDir/Rocky/rocky4/Rocky`) and it will ask how to point to your license server. 

If you did not set it up, you can run the following commands and substitute the ip address and port of your choosing. 
```
mkdir ~/.Rocky/
echo SERVER 192.168.0.1 ANY 1515 > ~/.Rocky/license.lic
echo USE_SERVER >> ~/.Rocky/license.lic
echo license_id: FlexLM > ~/.Rocky/license_definition.txt
echo license_path: /home/opc/.Rocky/license.lic >> ~/.Rocky/license_definition.txt
echo version: 1 >> ~/.Rocky/license_definition.txt
```

## Running the Application
Running Rocky as pretty straightforward: 
You can either start the GUI if you have a VNC session started with 
```
rockyHomeDir=/mnt/disk1/
$rockyHomeDir/Rocky/rocky4/Rocky
```

if you do not, you can run Rocky in batch mode:

Example 1 use 2 GPUs for modelName
```
$rockyHomeDir/Rocky/rocky4/Rocky --simulate modelName --resume 0 --use-gpu 1 --gpu-num=0 --gpu-num=1 
```
Example 2 use 32 CPUs for modelName
```
$rockyHomeDir/Rocky/rocky4/Rocky --simulate modelName --resume 0 --use-gpu 0 -ncpus 32
```
