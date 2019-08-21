# <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/rockylogo.png" height="40"> Runbook


# Deployment via web console

**Table of Contents**

- [Deployment via web console](#deployment-via-web-console)
  - [Log In](#log-in)
  - [Virtual Cloud Network](#virtual-cloud-network)
  - [Compute Instance](#compute-instance)
  - [Block Storage](#block-storage)
  - [Mounting a drive](#mounting-a-drive)
  - [Creating a Network File System](#creating-a-network-file-system)
    - [Headnode](#headnode)
    - [Visualization Node](#visualization-node)
  - [Adding a GPU Node for pre/post processing](#adding-a-gpu-node-for-prepost-processing)
  - [Set up a VNC](#set-up-a-vnc)
  - [Accessing a VNC](#accessing-a-vnc)
- [Installation](#installation)
  - [Connecting all compute node](#connecting-all-compute-node)
  - [Configuring NVIDIA GPUs](#configuring-nvidia-gpus)
  - [Installing Rocky DEM](#installing-rocky-dem)
  - [Licensing](#licensing)

## Log In
You can start by logging in the Oracle Cloud console. If this is the first time, instructions to do so are available [here](https://docs.cloud.oracle.com/iaas/Content/GSG/Tasks/signingin.htm).
Select the region in which you wish to create your instance. Click on the current region in the top right dropdown list to select another one. 

<img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/Region.png" height="50">

## Virtual Cloud Network
Before creating an instance, we need to configure a Virtual Cloud Network. Select the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/menu.png" height="20"> on the top left, then select Networking and Virtual Cloud Networks. <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/create_vcn.png" height="20">

On the next page, select the following: 
* Name of your VCN
* Compartment of your VCN
* Choose "CREATE VIRTUAL CLOUD NETWORK PLUS RELATED RESOURCES"

Scroll all the way down and <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/create_vcn.png" height="20">

Close the next window. 

## Compute Instance
Create a new instance by selecting the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/menu.png" height="20"> on the top left, then select Compute and Instances. 

<img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/Instances.png" height="300">

On the next page, select <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/create_instance.png" height="25">

On the next page, select the following:
* Name of your instance
* Availibility Domain: Each region has multiple availability domain. Some instance shapes are only available in certain AD.
* Change the image source to CentOS 7 or Oracle Linux 7.6.
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

## Block Storage

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

## Mounting a drive

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


## Creating a Network File System

In case you want to have a visualization node that is different from your main compute node/ head node. You should share the drive between the machines. 

### Headnode

Since the headnode is in a public subnet, we will keep the firewall up and add an exception through. 
```
sudo firewall-cmd --permanent --zone=public --add-service=nfs
sudo firewall-cmd --reload
```
We will also activate the nfs-server:

```
sudo yum -y install nfs-utils
sudo systemctl enable nfs-server.service
sudo systemctl start nfs-server.service
```

Edit the file /etc/exports with vim or your favorite text editor. `sudo vi /etc/exports` and add the line `/mnt/share   10.0.0.0/16(rw)`

To activate those changes:

```
sudo exportfs -a
```

### Visualization Node

We will also install the nfs-utils tools and mount the drive. You will need to grab the private IP address of the headnode. You can find it in the instance details in the webbrowser where you created the instances, or find it by running the command `ifconfig` on the headnode. It will probably be something like 10.0.0.2, 10.0.1.2 or 10.0.2.2 depending on the CIDR block of the public subnet. 

```
sudo firewall-cmd --permanent --zone=public --add-service=nfs
sudo firewall-cmd --reload
sudo yum -y install nfs-utils
sudo mkdir /mnt/share
sudo mount 10.0.0.2:/mnt/share /mnt/share
```

## Adding a GPU Node for pre/post processing

Rocky DEM can let you take advantage of the power of GPUs for post-processing your model. We can turn a GPU node on demand while the simulation is done. 

Create a new instance by selecting the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/menu.png" height="20"> on the top left, then select Compute and Instances. 

<img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/Instances.png" height="300">

On the next page, select <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/create_instance.png" height="25">

On the next page, select the following:
* Name of your instance
* Availibility Domain: Each region has multiple availability domain. Some instance shapes are only available in certain AD.
* Change the image source to Oracle Linux 7.6
* Instance Type: Select Bare metal for BM.GPU2.2 or Virtual Machine for VM.GPU2.1
* Instance Shape: 
  * BM.GPU2.2
  * VM.GPU2.1
  * BM.GPU3.8
  * VM.GPU3.*
  * Other shapes are available as well, [click for more information](https://cloud.oracle.com/compute/bare-metal/features).
* SSH key: Attach your public key file. [For more information](https://docs.cloud.oracle.com/iaas/Content/GSG/Tasks/creatingkeys.htm).
* Virtual Cloud Network: Select the network that you have previsouly created. In the case of a cluster: Select the public subnet.

Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/create_instance.png" height="20">

After a few minutes, the instances will turn green meaning it is up and running. You can now SSH into it. After clicking on the name of the instance, you will find the public IP. You can now connect using `ssh opc@xx.xx.xx.xx` from the machine using the key that was provided during the creation. 

Use SSH to remote login to the machine and mount the share drive as show before: 

```
sudo firewall-cmd --permanent --zone=public --add-service=nfs
sudo firewall-cmd --reload
sudo yum -y install nfs-utils
sudo mkdir /mnt/share
sudo mount 10.0.0.2:/mnt/share /mnt/share
```

You will need to follow the steps to set up a VNC session described below. 


## Set up a VNC
If you used terraform to create the cluster, this step has been done already for the GPU instance.

By default, the only access to the machines is through SSH in a console mode. If you want to see the Rocky DEM interface, you will need to set up a VNC connection. The following script will work for the default user opc. The password for the vnc session is set as "password" but it can be edited in the next commands. 

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
sudo systemctl start vncserver@:0.service
sudo systemctl enable vncserver@:0.service
```

## Accessing a VNC
We will connect through an SSH tunnel to the instance. On your machine, connect using ssh 

```
ssh -x -L 5902:127.0.0.1:5900 opc@public_ip
```

You can now connect using any VNC viewer using localhost:2 as VNC server and the password you set during the vnc installation. 

If you would rather connect without a SSH tunnel. You will need to open ports 5900 and 5901 on the Linux machine both in the firewall and in the security list. 

```
sudo firewall-offline-cmd --zone=public --add-port=5900-5901/tcp
```

Select the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/menu.png" height="20"> on the top left, then select Networking and Virtual Cloud Networks. <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/create_vcn.png" height="20">

Select the VCN that you created. Select the Subnet in which the machine reside, probably your public subnet. Select the security list. 

Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/addIngress.png" height="20">  

* CIDR : 0.0.0.0/0
* IP PROTOCOL: TCP
* Source Port Range: All
* Destination Port Range: 5900-5901
Click <img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/addIngress.png" height="20"> 

Now you should be able to VNC to the address: ip.add.re.ss:5900

Once you accessed your VNC session, you should go into Applications, then System Tools Then Settings.  

<img src="https://github.com/oci-hpc/oci-hpc-runbook-shared/raw/master/images/CentOSSeetings.jpg" height="100"> 

In the power options, set the Blank screen timeout to "Never". If you do get locked out of your user session, you can ssh to the instance and set a password for the opc user. 

```
sudo passwd opc
```

# Installation
This guide will show the different steps for the Oracle Linux 7.6 and CentOS 7 images available on Oracle Cloud Infrastructure. 
There is no need to configure NVIDIA GPUs if you have selected an Oracle LINUX version for GPU.

## Configuring NVIDIA GPUs
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

## Installing Rocky DEM
There are a couple of library that need to be added to the CentOS image. 
```sudo yum -y install mesa-libGLU-devel mesa-libGL-devel```

You can download the Rocky installer using wget from the ESSS website or other place. In our architecture, we have mounted a file system in the /mnt directory.

```
rockyHomeDir=/mnt/disk1
sudo mkdir $rockyHomeDir/Rocky
sudo chmod 777 $rockyHomeDir/Rocky
sudo tar -jxvf rocky4-bin-4.2.0-linux64.tbz2 -C $rockyHomeDir/Rocky/
```

If you change the variable for the rocky installer package in the terraform or resource manager, the package will be extracted on the selected drive. NVMe, block or FSS. 

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

There is a variable in the terraform stack that you can edit with the license path and port and it will do that step for you. 
