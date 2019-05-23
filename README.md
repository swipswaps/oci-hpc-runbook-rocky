# <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/rockylogo.png" height="40"> Runbook

## Introduction
This Runbook will take you through the process of deploying a GPU machine on Oracle Cloud Infrastructure, installing Rocky DEM, configuring the license, and then running a model.

Rocky DEM simulates particles and can be coupled with Computational Fluid Dynamic or Finite Element Method. It adds another layer of complexity which results in increased simulation time. To help speed up the run, Rocky DEM provides the option to parallelize to a high number of CPU. For even more speed, it can unleash the full power of one or multiple GPUs. 

Running Rocky on Oracle Cloud Infrastructure, there is no special setup needed and no driver headache. Import your model, choose the number of CPUs or GPUs and off you go. This removes the wait time for resources that you may have on your on-premise cluster. It avoids having people battling for high-end GPUs at peak times and having them idle for the rest of the week.

![](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/3184v0.gif "Example DEM simulation in Rocky")
## Architecture
The architecture for this runbook is simple, a single GPU machine running inside of an OCI VCN.

![](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/HPC_arch_draft.png "GPU Architecture for Running Rocky in OCI")
## Deployment
There are multiple options available to get started with Rocky on OCI. The next 2 sections will show how to do it from the console in a webbrowser and using a Terraform script. Scripts are especially usefull with more complex architecture.    
### Console
You can start by logging in the Oracle Cloud console. If this is the first time, instructions to do so are available [here](https://docs.cloud.oracle.com/iaas/Content/GSG/Tasks/signingin.htm).
Create a new instance by selecting the menu <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/Menu.png" height="5"> on the top left, then select Compute and Instances. 
<img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/Instances.png" height="50">


### Terraform Script
## Installation
This guide will show the different steps for the CentOS 7 image available on Oracle Cloud Infrastructure. 
### Configuring Nvidia GPUs
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
`sudo yum -y install mesa-libGLU-devel mesa-libGL-devel`

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

Now, you can use your favorite VNC viewer to log on and see the desktop.
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
`$rockyHomeDir/Rocky/rocky4/Rocky`

if you do not, you can run Rocky in batch mode:

Example 1 use 2 GPUs for mmodelName
```
$rockyHomeDir/Rocky/rocky4/Rocky --simulate modelName --resume 0 --use-gpu 1 --gpu-num=0 --gpu-num=1 
```
Example 2 use 32 CPUs for mmodelName
```
$rockyHomeDir/Rocky/rocky4/Rocky --simulate modelName --resume 0 --use-gpu 0 -ncpus 32
```

## Post-processing

## Expected Results
