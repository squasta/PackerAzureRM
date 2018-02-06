# PackerAzureRM - Examples to create Azure VM Images with Packer (en-us) 

Packer is an open source tool for creating identical machine images for multiple platforms from a single source configuration. Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel. Packer does not replace configuration management like Chef or Puppet. In fact, when building images, Packer is able to use tools like Chef or Puppet to install software onto the image. Packer only builds images. It does not attempt to manage them in any way. After they're built, it is up to you to launch or destroy them as you see fit.

* Read this in [french](readme.fr.md)
--------------------------------------------------------------------------------------------------------
The purpose of this repositary is to help you to start using Packer and Terraform to build your custom VM images on Microsoft Azure Cloud.

![Terraform + Azure + Packer Logo](https://github.com/squasta/PackerAzureRM/blob/master/AzurePackerTerraform.PNG)

--------------------------------------------------------------------------------------------------------

**Step 1 : Download and install Packer & Terraform**

Packer binaries are available here  : </br>[https://www.packer.io/downloads.html](https://www.packer.io/downloads.html)

Terraform binaries are avaible here : </br>[https://www.terraform.io/downloads.html](https://www.terraform.io/downloads.html)

> Don't forget to put in PATH of your operating system the location of Packer & Terraform

---------------------------------------------------------------------------------------------------------

**Step 2 : Prepare Azure prerequisite to connect Packer to Microsoft Azure**

You can connect Packer to Azure using your Azure Credential but it's better (my opinion) to use an Azure Service Principal Name (SPN).
You need first an Azure Active Directory, then 3 ways to do that : 
- Use portal to create an Azure Active Directory application and service principal that can access resources : </br>[https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal)
- Create an Azure service principal with Azure PowerShell : </br>
[https://docs.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azurermps-4.2.0](https://docs.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azurermps-4.2.0)
- Create an Azure service principal with Azure CLI 2.0 : </br>[https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager%2ftoc.json](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager%2ftoc.json)

---------------------------------------------------------------------------------------------------------

**Step 3 : Create Azure Infrastructure components that Packet needs to create a VM during the build Phase then for testing a deployment of the image**

Packer needs : 
- a resource group
- an Azure VNet
- a Subnet
- a Network Interface 
- a Public IP
- a Storage Account (if you want the final image in a storage account. Not necessary for managed disk)

> You can create these components using Web Portal, Azure PowerShell, Azure CLI, an ARM Template or Terraform (it's my choice here.)

3 Terraform File are available here :
- ``1-ConnexionAzure.tf`` : contains SPN informations to connect Terraform to Microsoft Azure. You need to modify this file with your SPN info.
- ``2-RG-StorageAccount.tf`` : defintion of a Resource Group and an Azure Storage Account
- ``3-PrerequisNetworkAzurepourVMdeployeeparPacker.tf`` : definition of all Azure IaaS component to deploy a VM using the Packer generated VM image

---------------------------------------------------------------------------------------------------------

**Step 4 : Create a JSON file for Packer**

JSON File section used by Packer are the following: 
- ``"variables": ["..."],``        ==> variables
- ``"builders": ["..."],``         ==> provider where image is build, Connection information, VM size to prepare image...
- ``"provisioners": ["..."]``      ==> add functionnalities, install applications, execute configuration scripts, Configuration Manager tools (Chef, Puppet, Ansible..)...

All Packer-name.json files in this repo are example you can use and adapt.

>/!\ Attention : there are difference between a JSON for a Linux VM and a JSON for a Windows VM: for Windows VM you need to provide the SPN ObjectID 
>>==> To obtain this ObjectID, use :
>>- the following Powershell command with SPN name : Get-AzureRmADServicePrincipal -SearchString "NameofyourSPN"
>>- or the Azure CLI 2.0 command (Thanks to **Etienne Deneuve** who provides me this command):
>>az ad sp list --query "[?displayName=='NameofyourSPN'].{ name: displayName, objectId: objectId }" <br/>
>>
>> **You can now use also : ``az ad sp list --spn http://Packer``**
>>```Bash
>>az ad sp list --spn http://Packer
>>[
>>  {
>>    "appId": "852e8410-5762-4890-XXXX-36b3f002377c",
>>    "displayName": "Packer",
>>    "objectId": "a5a66fab-913a-455a-XXXX-d0eb534cb90e",
>>    "objectType": "ServicePrincipal",
>>    "servicePrincipalNames": [
>>      "http://Packer",
>>      "852e8410-5762-4890-XXXX-36b3f002377c"
>>    ]
>>  }
>>]
>>```
- To check the name of a SPN with an ObjectID : `` az ad sp show --id XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXXX  <-- here objectid of Service Principal Name.`` Note : the az command can take many minutes to provide a result, depending on the size of your Azure AD. PowerShell command is really quicker (filtering is done on server side)



---------------------------------------------------------------------------------------------------------

**Step 5: Validate your JSON fichier with Packer**

When your Packer's JSON file is ready, validate it with the following Packer command : 
``packer validate nameofjsonfileforPacker.json``

---------------------------------------------------------------------------------------------------------
**Step 6 : Build your image in Azure with Packer.**

Execute the following command : ``packer build nameofjsonfileforPacker.json``

The basic steps performed by Packer to create a Linux image build are:
1. Create a resource group.
1. Validate and deploy a VM template.
1. Execute provision - defined by the user; typically shell commands.
1. Power off and capture the VM.
1. Delete the resource group.
1. Delete the temporary VM's OS disk.

The basic steps performed by Packer to create a Windows image build are:
1. Create a resource group.
1. Validate and deploy a KeyVault template.
1. Validate and deploy a VM template.
1. Execute provision - defined by the user; typically shell commands.
1. Power off and capture the VM.
1. Delete the resource group.
1. Delete the temporary VM's OS disk.

The output from the Packer build process is a virtual hard disk (VHD) in the specified storage account or an image disk (Azure Managed Disk) depending on your choice in Packer's JSON file. Packer also generate an ARM Template file in JSON.

---------------------------------------------------------------------------------------------------------
**Step 7 : Create a new VM in Azure based on the image built by Packer**

Download and customize ARMdeploy.ps1 with the ARM file generated by Packer
Execute the ARMdeploy.ps1 script

---------------------------------------------------------------------------------------------------------
**Global Summary** 

The following step must be follow to create a custom image in Azure:
1. Download and Install Terraform & Packer
1. in a folder, copy and customize .tf file, Packer-XXXXXX.json, fichierparametres.parameters.json, ARMDeploy.ps1
1. Execute Terraform to build resource group and Azure resources (Nic, VM, Storage...): ``Terraform apply``
1. Validate Packer package. Example: ``packer validate Packer-VMRHEL73StanAzureCustom.json``
1. Build Packer package. Example: ``packer build -color=true Packer-VMRHEL73StanAzureCustom.json``
1. Download ARM JSON file created by Packer and rename it. Example: ARM-VMRHEL73StanAzureCustom.json   (This ARM template is generated as an artifact if you are building an image (vhd file) in a storage account. If you are creating a disk managed image then you have to generate your our arm template or customize one that you get on Azure Quickstart template Github) 
1. Modify parameters.JSON file with NIC ID (you can get this ID with Terraform output command): fichierparametres.parameters.json
1. Execute Powershell script:  ``.\ARMDeploy.ps1``

Follow these steps to destroy and clean you environment:
1. Delete VM Generated from Packer image
1. ``Terraform Destroy``
