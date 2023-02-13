# Automating Azure Image Pipelines with HCP Packer

This repo is based on the Packer and Terraform code used in the [DevOps Lab](https://learn.microsoft.com/en-us/shows/devops-lab/?terms=hashicorp) episode "Automating Azure Image Pipelines with HCP Packer".  That original repo has been split into two separate repos, this one for the image workflow and a second repo for [deploying HashiCafe to Azure](https://github.com/HCDemos/hashicafe-Azure-HCP-PackerVault).  Both repos have been enhanced to leverage dynamic Azure credentials using HCP Vault, which was originally configured by following the directions in the learn [Azure Secrets Engine tutorial](https://developer.hashicorp.com/vault/tutorials/secrets-management/azure-secrets).  For this repo a Github Actions pipeline has been configured so that updates to the repo will trigger image updates.  Since the image updates take some time (~13minutes for each) the actual image that gets updated is controlled by updating the reference in the pipeline yaml file (so you can build the base image version before a demo and perhaps the child image during the demo - but have other things to talk about...)

![Alt text](/images/image-workflow-packer-hcp-packer-vault-azure.png "HCP Vault - TFCB - Github Actions Workflow")

Below are the original instructions for updates required for Azure/HCP Packer to enable them to work with this repo.   
These configurations demonstrate using Packer to build customized Azure VM images which are published to the HCP Packer hosted image registry and consumed by a Terraform provisioning workflow. Image ancestry tracking in HCP Packer is also demonstrated by building a parent/child image pipeline.

## Requirements

- An Azure subscription with an existing Resource Group where builds will occur and images will be stored, including a Compute Gallery pre-populated as described in the Notes section below
- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/)
- An [HCP Packer](https://cloud.hashicorp.com/products/packer) image registry (free for up to 10 images)
- [Packer](https://www.packer.io/) 1.7.7 or newer
- [Terraform](https://www.terraform.io/) 1.0 or newer

## Contents

- `hcp-packer-ubuntu20` - the base or "parent" image
- `hcp-packer-ubuntu20-nginx` - "child" image which derives from the base to add Nginx
- `hashicafe-webserver` - a Terraform configuration to deploy the image as a simple webapp

## Notes

The Packer builds will create a managed image and also publish to an [Azure compute gallery](https://learn.microsoft.com/en-us/azure/virtual-machines/azure-compute-gallery), which must be located in the same Resource Group as your images.

The compute gallery must have VM image definitions pre-created with the following settings:

- VM image definition name: `ubuntu20-base` and `ubuntu20-nginx`
- OS type: Linux
- VM generation: Gen2
- Publisher/Offer/SKU: any valid values are fine

Optionally, you can remove the destination_shared_image_gallery block from the Packer builds; it's included to demonstrate the capability but isn't critical to the workflow.

## Steps

1. Sign in to your Azure account with `az login`.
2. Make a copy of the `packer.auto.pkrvars.hcl.example` and `terraform.auto.tfvars.example` files without the `.example` suffix, and fill in your desired values. See each variables file for descriptions.
3. Run the `hcp-packer-ubuntu20` build, then in HCP Packer assign the new iteration in the **ubuntu20-base** bucket to a channel named "development".
4. Run the `hcp-packer-ubuntu20-nginx` build, then assign the new iteration in the **ubuntu20-nginx** bucket to a channel named "development".
5. Run the `hashicafe-webserver` Terraform configuration.
