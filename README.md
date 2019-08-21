# <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/rockylogo.png" height="40"> Runbook

# Introduction
This Runbook will take you through the process of deploying a GPU machine on Oracle Cloud Infrastructure, installing Rocky DEM, configuring the license, and then running a model.

Rocky DEM simulates particles and can be coupled with Computational Fluid Dynamic or Finite Element Method. It adds another layer of complexity which results in increased simulation time. To help speed up the run, Rocky DEM provides the option to parallelize to a high number of CPU. For even more speed, it can unleash the full power of one or multiple GPUs. 

Running Rocky on Oracle Cloud Infrastructure, there is no special setup needed and no driver headache. Import your model, choose the number of CPUs or GPUs and off you go. This removes the wait time for resources that you may have on your on-premise cluster. It avoids having people battling for high-end GPUs at peak times and having them idle for the rest of the week.

![](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/3184v0.gif "Example DEM simulation in Rocky")
 
**Table of Contents**
- [Introduction](#introduction)
- [Architecture](#architecture)
- [Deployment through Resource Manager](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/Documentation/ResourceManager.md#deployment-through-resource-manager)
- [Deployment through Terraform Script](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/Documentation/terraform.md#deployment-through-terraform-script)
- [Deployment via web console](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/Documentation/ManualDeployment.md#deployment-via-web-console)
- [Installing Rocky](#installing-rocky)
- [Running the Application](#running-the-application)
- [Benchmark Example](#benchmark-example)
  - [17 Millions Cells](#17-millions-cells)
  - [105 Millions Cells](#105-millions-cells)
 
# Architecture
The architecture for this runbook is as follow, we have one main machine (The headnode) that will start the jobs. Other machines (Compute Nodes) will be accessible from the headnode and STAR-CCM+ will distribute the jobs to the compute nodes. The headnode will be accesible through SSH from anyone with the key (or VNC if you decide to enable it) Compute nodes will only be accessible from inside the network. This is made possible with 1 Virtual Cloud Network with 2 subnets, one public and one private.   

![](https://github.com/oci-hpc/oci-hpc-runbook-shared/blob/master/images/HPC_arch_draft.png "GPU Architecture for Running Rocky in OCI")

# Deployment

Deploying this architecture on OCI can be done in different ways.
* The [resource Manager](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/Documentation/ResourceManager.md#deployment-through-resource-manager) let you deploy it from the console. Only relevant variables are shown but others can be changed in the zip file. 
* [Terraform](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/Documentation/terraform.md#terraform-installation) is a scripting language for deploying resources. It is the foundation of the Resource Manager, using it will be easier if you need to make modifications to the terraform stack often. 
* The [web console](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/Documentation/ManualDeployment.md#deployment-via-web-console) let you create each piece of the architecture one by one from a webbrowser. This can be used to avoid any terraform scripting or using existing templates. 

# Running the Application
Running Rocky as pretty straightforward: 
You can either start the GUI if you have a VNC session started with 
```
rockyHomeDir=/mnt/block/
$rockyHomeDir/Rocky/rocky4/Rocky
```

If you do not, you can run Rocky in batch mode:

Example 1 use 2 GPUs for modelName
```
$rockyHomeDir/Rocky/rocky4/Rocky --simulate modelName --resume 0 --use-gpu 1 --gpu-num=0 --gpu-num=1 
```
Example 2 use 32 CPUs for modelName
```
$rockyHomeDir/Rocky/rocky4/Rocky --simulate modelName --resume 0 --use-gpu 0 -ncpus 32
```

# Benchmark Example
## 
## 
