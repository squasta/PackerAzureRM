# PackerAzureRM - Examples to create Azure VM Images with Packer (en-us) - Exemples pour créer des images de VM dans Azure avec Packer (fr)

Packer is an open source tool for creating identical machine images for multiple platforms from a single source configuration. Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel. Packer does not replace configuration management like Chef or Puppet. In fact, when building images, Packer is able to use tools like Chef or Puppet to install software onto the image. Packer only builds images. It does not attempt to manage them in any way. After they're built, it is up to you to launch or destroy them as you see fit.

Packer est un outil Open Source permettant de créer des images de machines virtuelles sur plusieurs plateforme depuis une configuration source unique. Packer est un outil assez léger et très puissant de création d'image de machines. Il n'a pas pour objectif de remplacer des outils de Configuration Management comme Chef, Puppet ou Ansible. En fait, lors de la construction d'une image, il est capable d'utiliser ces outils de CM pour installer des logiciels dans l'image. Packer ne sert qu'à construire des images et ne permet pas leur gestion ultérieure. Une fois construites, c'est à vous de les instancier ou de les détruire.

--------------------------------------------------------------------------------------------------------
The purpose of this repositary is to help you to start using Packer and Terraform to build your custom VM images on Microsoft Azure Cloud.

L'objectif de ce dépôt est de vous aider à commencer d'utiliser Packer et Terraform pour construire des images de machines virtuelles personnalisées pour le Cloud Microsoft Azure.

![alt text](https://github.com/squasta/PackerAzureRM/blob/master/AzurePackerTerraform.PNG)

--------------------------------------------------------------------------------------------------------

**Step 1 : Download and install Packer & Terraform**
Packer binaries are available here  : https://www.packer.io/downloads.html
Terraform binaries are avaible here : https://www.terraform.io/downloads.html
Don't forget to put in PATH of your operating system the location of Packer & Terraform

**Etape 1 : Télécharger et installer Packer & Terraform**
Les binaires de Packer sont disponibles ici : https://www.packer.io/downloads.html
Les binaires de Terraform sont disponibles ici : https://www.terraform.io/downloads.html
Ne pas oublier de mettre dans le PATH de votre système d'exploitation le chemin où se trouvent Packer et Terraform

---------------------------------------------------------------------------------------------------------

**Step 2 : Prepare Azure prerequisite to connect Packer to Microsoft Azure** 
You can connect Packer to Azure using your Azure Credential but it's better (my opinion) to use an Azure Service Principal Name (SPN).
You need first an Azure Active Directory, then 3 ways to do that : 
- Use portal to create an Azure Active Directory application and service principal that can access resources : https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal
- Create an Azure service principal with Azure PowerShell : https://docs.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azurermps-4.2.0
- Create an Azure service principal with Azure CLI 2.0 : https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager%2ftoc.json

**Etape 2 : Préparer les pré-requis Azure pour connecter Packer à Microsoft Azure**
Vous pouvez connecter Packer à Azure en utilisant vos informations de sécurité mais c'est mieux (à mon avis) d'utiliser un Azure Service Principal Name (SPN).
Cela nécessite l'existence d'un Azure Active Directory
J'ai documenté la création de ce SPN dans un article en français ici : https://stanislas.io/2017/01/02/modeliser-deployer-et-gerer-des-ressources-azure-avec-terraform-de-hashicorp/

---------------------------------------------------------------------------------------------------------

**Step 3 : Create Azure Infrastructure components that Packet needs to create a VM during the build Phase then for testing a deployment of the image**
Packer needs : 
- a resource group
- an Azure VNet
- a Subnet
- a Network Interface 
- a Public IP
- a Storage Account (if you want the final image in a storage account. Not necessary for managed disk)

You can create these components using Web Portal, Azure PowerShell, Azure CLI, an ARM Template or Terraform (it's my choice here.)

3 Terraform File are available here :
- 1-ConnexionAzure.tf : contains SPN informations to connect Terraform to Microsoft Azure. You need to modify this file with your SPN info.
- 2-RG-StorageAccount.tf : defintion of a Resource Group and an Azure Storage Account
- 3-PrerequisNetworkAzurepourVMdeployeeparPacker.tf : definition of all Azure IaaS component to deploy a VM using the Packer generated VM image

**Etape 3 : créer les éléments d'infrastructure dans Azure nécessaires à Packer pour créer sa VM lors de la phase de Build et nécessaire pour tester un déploiement de l'image créée**
Packer a besoin de :
- un groupe de ressource
- un VNet Azure
- un Subnet
- une carte réseau 
- une IP Publique 
- un compte de stockage Azure (si vous voulez l'image finale dans un compte de stockage. Non nécessaire pour les disques managés)

Il est possible de faire cela via le portail Azure, Azure PowerShell, la ligne de commande azure (az), un modèle ARM ou via Terraform (c'est le choix que j'ai privilégié).

3 fichiers Terraform File sont disponibles ici :
- 1-ConnexionAzure.tf : contient les informations du SPN permettant à Terraform d'agir sur un abonnement Azure. Vous devez modifier ce fichier avec les informations de votre SPN.
- 2-RG-StorageAccount.tf : Définition d'un groupe de ressources et d'un compte de stockage Azure.
- 3-PrerequisNetworkAzurepourVMdeployeeparPacker.tf : définition de toutes les composants IaaS d'Azure pour déployer une VM depuis l'image générée par Packer

---------------------------------------------------------------------------------------------------------

**Step 4 : Create a JSON file for Packer**

JSON File section used by Packer are the following: 
- "variables": ["..."],        ==> variables
- "builders": ["..."],         ==> provider where image is build, Connection information, VM size to prepare image...
- "provisioners": ["..."]      ==> add functionnalities, install applications, execute configuration scripts, Configuration Manager tools (Chef, Puppet, Ansible..)...

All Packer-name.json files in this repo are example you can use and adapt.

/!\ Attention : there are difference between a JSON for a Linux VM and a JSON for a Windows VM: for Windows VM you need to provide the SPN ObjectID 
==> To obtain this ObjectID, use :
- the following Powershell command with SPN name : Get-AzureRmADServicePrincipal -SearchString "NameofyourSPN"
- or the Azure CLI 2.0 command (Thanks to Etienne Deneuve who provides me this command):
az ad sp list --query "[?displayName=='NameofyourSPN'].{ name: displayName, objectId: objectId }"
- To check the name of a SPN with an ObjectID : az ad sp show --id XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXXX  <-- here objectid of Service Principal Name
Note : the az command can take many minutes to provide a result, depending on the size of your Azure AD. PowerShell command is really quicker (filtering is done on server side)


**Etape 4 : Créer un fichier JSON pour Packer**

Les sections d'un fichier JSON utilisé par Packer pour créer une image sont les suivantes : 
- "variables": ["..."],        ==> variables
- "builders": ["..."],         ==> choix du provider pour lequel l'image est crée, informations de connexion, taille de a VM à créer pour préparer l'image...
- "provisioners": ["..."]      ==> ajout de fonctionnalités, installation d'applications, déclenchement de scripts de configuration, appel à des outils de Configuration Manager (Chef, Puppet, Ansible..)...
/!\ don't forget to customize builder section with your SPN, ObjectID and Storage Account Name

Tous les fichiers Packer-name.json de ce répertoire sont des exemples que vous pouvez utiliser et adapter.
Attention à ne pas oublier de personnaliser ces fichiers avec vos informations : SPN, ObjectID SPN, orget to customize builder section with your SPN, Nom du compte de stockage

Note importante : parmi les différences entre un JSON pour créer une VM Linux et un JSON pour créer une image Windows, il y a la nécessiter de fournir pour l'image Windows l'ObjectID du SPN (Service Principal Name)
==> Pour obtenir cet ObjectID, utiliser :
- la commande Powershell suivante avec le nom du SPN : Get-AzureRmADServicePrincipal -SearchString "NameofyourSPN"
- ou la commande az suivante (Merci à Etienne Deneuve qui en est l'auteur):
az ad sp list --query "[?displayName=='NameofyourSPN'].{ name: displayName, objectId: objectId }"
- Pour vérifier le nom d'un SPN avec un ID : az ad sp show --id XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXXX  <-- ici l'objectid du ServicePrincipal

---------------------------------------------------------------------------------------------------------

**Step 5: Validate your JSON fichier with Packer**

When your Packer's JSON file is ready, validate it with the following Packer command : 
packer validate nameofjsonfileforPacker.json

Etape 5 : Valider le fichier JSON pour Packer

Une fois ce fichier correctement écrit, le valider avec la commande :
packer validate nomdufichierjsonpourPacker.json

---------------------------------------------------------------------------------------------------------
**Step 6 : Build your image in Azure with Packer.**

Execute the following command : packer build nameofjsonfileforPacker.json

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

**Etape 6 : Faire construire l'image dans Azure par Packer** 

Pour cela on va utiliser l'option build : packer build nomdufichierjsonpourPacker.json

Les étapes basiques effectuée par Packer suite à l'exécution de cette commande sont pour une image Linux :
1. Création d'un ressource group
1. Validation et déploiement d'un modèle de VM
1. Exécution du provisionnement de la VM (typiquement des commandes shell)
1. Arrêt et capture de la VM
1. Suppression du ressource group créé en 1
1. Suppression du disque OS de la VM créée en 2

Les étapes basiques effectuée par Packer suite à l'exécution de cette commande sont pour une image Windows :
1. Création d'un ressource group
1. Validation et déploiement d'un modèle Azure KeyVault
1. Validation et déploiement d'un modèle de VM
1. Exécution du provisionnement de la VM (typiquement des commandes shell)
1. Arrêt et capture de la VM
1. Suppression du ressource group créé en 1
1. Suppression du disque OS de la VM créée en 2

Le résultat du processus de build de Packer est un disque dur virtuel (VHD) dans le compte de stockage spécifié ou une image (dans le cas d'utilisation d'un disque managé) en fonction du choix dans le fichier JSON Packer. Packer génère aussi un fichier de modèle ARM en JSON.


---------------------------------------------------------------------------------------------------------
**Step 7 : Create a new VM in Azure based on the image built by Packer**

Download and customize ARMdeploy.ps1 with the ARM file generated by Packer
Execute the ARMdeploy.ps1 script

**Etape 7 : Créer une nouvelle VM depuis l'image construite par Packer**

Télécharger et personnaliser le script PowerShell ARMdeploy.ps1, mettre les bons chemins vers le fichier de modèle ARM généré par Packer.
Exécuter le script ARMdeploy.ps1


---------------------------------------------------------------------------------------------------------
**Global Summary** 

The following step must be follow to create a custom image in Azure:
1. Download and Install Terraform & Packer
1. in a folder, copy and customize .tf file, Packer-XXXXXX.json, fichierparametres.parameters.json, ARMDeploy.ps1
1. Execute Terraform to build resource group and Azure resources (Nic, VM, Storage...): Terraform apply
1. Validate Packer package. Example: packer validate Packer-VMRHEL73StanAzureCustom.json
1. Build Packer package. Example: packer build -color=true Packer-VMRHEL73StanAzureCustom.json
1. Download ARM JSON file created by Packer and renameit. Example: ARM-VMRHEL73StanAzureCustom.json
1. Modify parameters.JSON file with NIC ID (you can get this ID with Terraform output command): fichierparametres.parameters.json
1. Execute Powershell script:  .\ARMDeploy.ps1

Follow these steps to destroy and clean you environnment:
1. Delete VM Generated from Packer image
1. Terraform Destroy


**Résumé Global :**

En résumé les étapes à suivre pour créer une image personnalisée dans Azure :
1. Télécharger et installer Terraform et Packer
1. Dans un répertoire, copier et personnaliser les fichiers .tf, le fichier Packer-XXXXX.json, ficierparametres.parameters.json, ARMDeploy.ps1
1. Lancer Terraform pour construire ressource group et créer les ressources nécesssaires (Nic, VM, Storage...) : Terraform apply
1. Valider le package Packer. Exemple : packer validate Packer-VMRHEL73StanAzureCustom.json
1. Builder le package Packer. Exemple: packer build -color=true Packer-VMRHEL73StanAzureCustom.json
1. Télécharger le modèle ARM JSON généré par Packer et le renommer ARM-VMRHEL73StanAzureCustom.json
1. Modifier le fichier parameters.json avec l'ID de la NIC créée par Terraform (visible via Terraform output) : fichierparametres.parameters.json
1. exécuter le script Powershell .\ARMDeploy.ps1

En résumé les étapes à suivre pour nettoyer votre environnement :
1. Supprimer la VM créée depuis l'image générée
1. Terraform Destroy
