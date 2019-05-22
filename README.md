# <img src="https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/rockylogo.png" height="80"> Runbook

## Introduction
This Runbook will take you through the process of deploying a GPU machine on Oracle Cloud Infrastructure, installing Rocky DEM, configuring the license, and then running a model.

Rocky DEM simulates particles and can be coupled with Computational Fluid Dynamic or Finite Element Method. It adds another layer of complexity which results in increased simulation time. To help speed up the run, Rocky DEM provides the option to parallelize to a high number of CPU. For even more speed, it can unleash the full power of one or multiple GPUs. 

Running Rocky on Oracle Cloud Infrastructure, there is no special setup needed and no driver headache. Import your model, choose the number of CPUs or GPUs and off you go. This removes the wait time for resources that you may have on your on-premise cluster. It avoids having people battling for high-end GPUs at peak times and having them idle for the rest of the week.

![](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/3184v0.gif "Example DEM simulation in Rocky")
## Architecture
The architecture for this runbook is simple, a single GPU machine running inside of an OCI VCN.

![](https://github.com/oci-hpc/oci-hpc-runbook-rocky/blob/master/images/HPC_arch_draft.png "GPU Architecture for Running Rocky in OCI")
## Deployment

## Installation

## Licensing

## Running the Application

## Post-processing

## Expected Results
