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

Step 3 : Create Azure Infrastructure components that Packet needs to create a VM during the build Phase
Packer needs : 
- a resource group
- an Azure VNet
- a Subnet
- a Network Interface 
- a Public IP
- a Storage Account (if you want the final image in a storage account. Not necessary for managed disk)

You can create these components using Web Portal, Azure PowerShell, Azure CLI, an ARM Template or Terraform (it's my choice here.)

Etape 3 : créer les éléments d'infrastructure dans Azure nécessaire à Packer pour créer sa VM lors de la phase de Build
Packer a besoin de :
- un groupe de ressource
- un VNet Azure
- un Subnet
- une carte réseau 
- une IP Publique 
- un compte de stockage Azure (si vous voulez l'image finale dans un compte de stockage. Non nécessaire pour les disques managés)

Il est possible de faire cela via le portail Azure, Azure PowerShell, la ligne de commande azure (az), un modèle ARM ou via Terraform (c'est le choix que j'ai privilégié).
