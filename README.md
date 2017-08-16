# PackerAzureRM - Examples to create Azure VM Images with Packer - Exemples pour créer des images de VM dans Azure avec Packer

Packer is an open source tool for creating identical machine images for multiple platforms from a single source configuration. Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel. Packer does not replace configuration management like Chef or Puppet. In fact, when building images, Packer is able to use tools like Chef or Puppet to install software onto the image. Packer only builds images. It does not attempt to manage them in any way. After they're built, it is up to you to launch or destroy them as you see fit.

Packer est un outil Open Source permettant de créer des images de machines virtuelles sur plusieurs plateforme depuis une configuration source unique. Packer est un outil assez léger et très puissant de création d'image de machines. Il n'a pas pour objectif de remplacer des outils de Configuration Management comme Chef, Puppet ou Ansible. En fait, lors de la construction d'une image, il est capable d'utiliser ces outils de CM pour installer des logiciels dans l'image. Packer ne sert qu'à construire des images et ne permet pas leur gestion ultérieure. Une fois construites, c'est à vous de les instancier ou de les détruire.

--------------------------------------------------------------------------------------------------------
The purpose of this repositary is to help you to start using Packer to build your custom VM images on Microsoft Azure Cloud.

L'objectif de ce dépôt est de vous aider à commencer d'utiliser Packer pour construire des images de machines virtuelles personnalisées pour le Cloud Microsoft Azure.

--------------------------------------------------------------------------------------------------------

Step 1 : Download and install Packer
Packer binaries are available here  : https://www.packer.io/downloads.html
Don't forget to put in PATH of your operating system the location of Packer

Etape 1 : Télécharger et installer Packer
Les binaires de Packer sont disponibles ici : https://www.packer.io/downloads.html
Ne pas oublier de mettre dans le PATH de votre système d'exploitation le chemin où se trouve Packer

---------------------------------------------------------------------------------------------------------

Step 2 : Prepare Azure prerequisite to connect Packer to Microsoft Azure 
You can connect Packer to Azure using your Azure Credential but it's better (my opinion) to use an Azure Service Principal Name (SPN).
You need first an Azure Active Directory, then 3 ways to do that : 
- Use portal to create an Azure Active Directory application and service principal that can access resources : https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal
- Create an Azure service principal with Azure PowerShell : https://docs.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azurermps-4.2.0
- Create an Azure service principal with Azure CLI 2.0 : https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager%2ftoc.json

Etape 2 : Préparer les pré-requis Azure pour connecter Packer à Microsoft Azure
Vous pouvez connecter Packer à Azure en utilisant vos informations de sécurité mais c'est mieux (à mon avis) d'utiliser un Azure Service Principal Name (SPN).
Cela nécessite l'existence d'un Azure Active Directory
J'ai documenté la création de ce SPN dans un article en français ici : https://stanislas.io/2017/01/02/modeliser-deployer-et-gerer-des-ressources-azure-avec-terraform-de-hashicorp/

---------------------------------------------------------------------------------------------------------

Step 3 : Create Azure Infrastructure components that Packet needs to create a VM during the build Phase then for testing a deployment of the image
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

Etape 3 : créer les éléments d'infrastructure dans Azure nécessaires à Packer pour créer sa VM lors de la phase de Build et nécessaire pour tester un déploiement de l'image créée
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

Step 4 : Create a JSON file for Packer

JSON File section used by Packer are the following: 
- "variables": ["..."],        ==> variables
- "builders": ["..."],         ==> provider where image is build, Connection information, VM size to prepare image...
- "provisioners": ["..."]      ==> add functionnalities, install applications, execute configuration scripts, Configuration Manager tools (Chef, Puppet, Ansible..)...

All Packer-name.json files in this repo are example you can use and adapt.

Etape 4 : Créer un fichier JSON pour Packer

Les sections d'un fichier JSON utilisé par Packer pour créer une image sont les suivantes : 
- "variables": ["..."],        ==> variables
- "builders": ["..."],         ==> choix du provider pour lequel l'image est crée, informations de connexion, taille de a VM à créer pour préparer l'image...
- "provisioners": ["..."]      ==> ajout de fonctionnalités, installation d'applications, déclenchement de scripts de configuration, appel à des outils de Configuration Manager (Chef, Puppet, Ansible..)...
/!\ don't forget to customize builder section with your SPN, ObjectID and Storage Account Name

Tous les fichiers Packer-name.json de ce répertoire sont des exemples que vous pouvez utiliser et adapter.
/!\ Ne pas outblier de personnaliser ces fichiers avec vos informations : SPN, ObjectID SPN, orget to customize builder section with your SPN, Nom du compte de stockage

---------------------------------------------------------------------------------------------------------

Step 5: Validate your JSON fichier with Packer

When your Packer's JSON file is ready, validate it with the following Packer command : 
packer validate nameofjsonfileforPacker.json

Etape 5 : Valider le fichier JSON pour Packer

Une fois ce fichier correctement écrit, le valider avec la commande :
packer validate nomdufichierjsonpourPacker.json

---------------------------------------------------------------------------------------------------------
Step 6 : Build your image in Azure with Packer.

Execute the following command : packer build nameofjsonfileforPacker.json

The basic steps performed by Packer to create a Linux image build are:
1- Create a resource group.
2- Validate and deploy a VM template.
3- Execute provision - defined by the user; typically shell commands.
4- Power off and capture the VM.
5- Delete the resource group.
6- Delete the temporary VM's OS disk.

The basic steps performed by Packer to create a Windows image build are:
1- Create a resource group.
2- Validate and deploy a KeyVault template.
3- Validate and deploy a VM template.
4- Execute provision - defined by the user; typically shell commands.
5- Power off and capture the VM.
6- Delete the resource group.
7- Delete the temporary VM's OS disk.

The output from the Packer build process is a virtual hard disk (VHD) in the specified storage account or an image disk (Azure Managed Disk) depending on your choice in Packer's JSON file

Etape 6 : Faire construire l'image dans Azure par Packer 

Pour cela on va utiliser l'option build : packer build nomdufichierjsonpourPacker.json

Les étapes basiques effectuée par Packer suite à l'exécution de cette commande sont pour une image Linux :
1- Création d'un ressource group
2- Validation et déploiement d'un modèle de VM
3- Exécution du provisionnement de la VM (typiquement des commandes shell)
4- Arrêt et capture de la VM
5- Suppression du ressource group créé en 1
6- Suppression du disque OS de la VM créée en 2

Les étapes basiques effectuée par Packer suite à l'exécution de cette commande sont pour une image Windows :
1- Création d'un ressource group
2- Validation et déploiement d'un modèle Azure KeyVault
3- Validation et déploiement d'un modèle de VM
4- Exécution du provisionnement de la VM (typiquement des commandes shell)
5- Arrêt et capture de la VM
6- Suppression du ressource group créé en 1
7- Suppression du disque OS de la VM créée en 2

Le résultat du processus de build de Packer est un disque dur virtuel (VHD) dans le compte de stockage spécifié ou une image (dans le cas d'utilisation d'un disque managé) en fonction du choix dans le fichier JSON Packer









============================================
En résumé :

En résumé les étapes à suivre pour créer une image personnalisée dans Azure :
1- Lancer le Terraform pour construire ressource group et créer les ressources nécesssaires (Nic, VM, Storage...) : Terraform apply
2- Valider le package Packer : packer validate Packer-VMRHEL73StanAzureCustom.json
3- Builder le package Packer : packer build -color=true Packer-VMRHEL73StanAzureCustom.json
4- Télécharger le modèle ARM JSON généré par Packer et le renommer Packer-VMRHEL73StanAzureCustom.json
5- Modifier le fichier avec l'ID de la NIC créée par Terraform (visible via Terraform output) : fichierparametres.parameters.json
6- exécuter le script Powershell ARMDeploy.ps1

En résumé les étapes à suivre pour nettoyer votre environnement :
1- Supprimer la VM créée depuis l'image générée
2- Terraform Destroy
