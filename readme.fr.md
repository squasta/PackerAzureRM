# PackerAzureRM - Exemples pour créer des images de VM dans Azure avec Packer (fr) 
<br/>

Packer est un outil Open Source permettant de créer des images de machines virtuelles sur plusieurs plateforme depuis une configuration source unique. Packer est un outil assez léger et très puissant de création d'image de machines. Il n'a pas pour objectif de remplacer des outils de Configuration Management comme Chef, Puppet ou Ansible. En fait, lors de la construction d'une image, il est capable d'utiliser ces outils de CM pour installer des logiciels dans l'image. Packer ne sert qu'à construire des images et ne permet pas leur gestion ultérieure. Une fois construites, c'est à vous de les instancier ou de les détruire.
* Read this in [english](readme.md)
--------------------------------------------------------------------------------------------------------
L'objectif de ce dépôt est de vous aider à commencer d'utiliser Packer et Terraform pour construire des images de machines virtuelles personnalisées pour le Cloud Microsoft Azure.

![Terraform + Azure + Packer Logo](https://github.com/squasta/PackerAzureRM/blob/master/AzurePackerTerraform.PNG)
--------------------------------------------------------------------------------------------------------
**Etape 1 : Télécharger et installer Packer & Terraform**

Les binaires de Packer sont disponibles ici : </br>[https://www.packer.io/downloads.html](https://www.packer.io/downloads.html)

Les binaires de Terraform sont disponibles ici : </br>[https://www.terraform.io/downloads.html](https://www.terraform.io/downloads.html)

> Ne pas oublier de mettre dans le PATH de votre système d'exploitation le chemin où se trouvent Packer et Terraform
---------------------------------------------------------------------------------------------------------
**Etape 2 : Préparer les pré-requis Azure pour connecter Packer à Microsoft Azure**</br>
Vous pouvez connecter Packer à Azure en utilisant vos informations de sécurité mais c'est mieux (à mon avis) d'utiliser un Azure Service Principal Name (SPN).
Cela nécessite l'existence d'un Azure Active Directory </br>
J'ai documenté la création de ce SPN dans un article en français ici : </br>[https://stanislas.io/2017/01/02/modeliser-deployer-et-gerer-des-ressources-azure-avec-terraform-de-hashicorp/](https://stanislas.io/2017/01/02/modeliser-deployer-et-gerer-des-ressources-azure-avec-terraform-de-hashicorp/)

Sinon, vous pouvez aussi consulter les documentations Microsoft (en Anglais):
- Via le portail Azure :  : </br>[https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal)
- Avec Azure PowerShell  : </br>
[https://docs.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azurermps-4.2.0](https://docs.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azurermps-4.2.0)
- Avec Azure CLI 2.0 : </br>[https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager%2ftoc.json](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager%2ftoc.json)

---------------------------------------------------------------------------------------------------------
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
- ``1-ConnexionAzure.tf`` : contient les informations du SPN permettant à Terraform d'agir sur un abonnement Azure. Vous devez modifier ce fichier avec les informations de votre SPN.
- ``2-RG-StorageAccount.tf`` : Définition d'un groupe de ressources et d'un compte de stockage Azure.
- ``3-PrerequisNetworkAzurepourVMdeployeeparPacker.tf`` : définition de toutes les composants IaaS d'Azure pour déployer une VM depuis l'image générée par Packer

---------------------------------------------------------------------------------------------------------

**Etape 4 : Créer un fichier JSON pour Packer**

Les sections d'un fichier JSON utilisé par Packer pour créer une image sont les suivantes : 
- ``"variables": ["..."],``        ==> variables
- ``"builders": ["..."],``         ==> choix du provider pour lequel l'image est crée, informations de connexion, taille de a VM à créer pour préparer l'image...
- ``"provisioners": ["..."]``      ==> ajout de fonctionnalités, installation d'applications, déclenchement de scripts de configuration, appel à des outils de Configuration Manager (Chef, Puppet, Ansible..)...
> /!\ don't forget to customize builder section with your SPN, ObjectID and Storage Account Name

Tous les fichiers Packer-name.json de ce répertoire sont des exemples que vous pouvez utiliser et adapter.
Attention à ne pas oublier de personnaliser ces fichiers avec vos informations : SPN, ObjectID SPN, orget to customize builder section with your SPN, Nom du compte de stockage

>>Note importante : parmi les différences entre un JSON pour créer une VM Linux et un JSON pour créer une image Windows, il y a la nécessiter de fournir pour l'image Windows l'ObjectID du SPN (Service Principal Name)
>>==> Pour obtenir cet ObjectID, utiliser :
>>- la commande Powershell suivante avec le nom du SPN : ``Get-AzureRmADServicePrincipal -SearchString "NameofyourSPN"``
>>- ou la commande az suivante (Merci à **Etienne Deneuve** qui en est l'auteur):
``az ad sp list --query "[?displayName=='NameofyourSPN'].{ name: displayName, objectId: objectId }"``
>> **Vous pouvez désormais utiliser : ``az ad sp list --spn http://Packer``**
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
>>- Pour vérifier le nom d'un SPN avec un ID : ``az ad sp show --id XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXXX  <-- ici l'objectid du ServicePrincipal.`` 
>>Attention l'exécution de la commande az peut prendre beaucoup beaucoup de temps (plusieurs minutes voire plus) en fonction de la taille de l'Azure Active Directory. La commande PowerShell est plus optimisée car le filtrage est fait côté serveur.

---------------------------------------------------------------------------------------------------------

**Etape 5 : Valider le fichier JSON pour Packer**

Une fois ce fichier correctement écrit, le valider avec la commande :
``packer validate nomdufichierjsonpourPacker.json``

---------------------------------------------------------------------------------------------------------
**Etape 6 : Faire construire l'image dans Azure par Packer** 

Pour cela on va utiliser l'option build : ``packer build nomdufichierjsonpourPacker.json``

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
**Etape 7 : Créer une nouvelle VM depuis l'image construite par Packer**

Télécharger et personnaliser le script PowerShell ARMdeploy.ps1, mettre les bons chemins vers le fichier de modèle ARM généré par Packer.
Exécuter le script ARMdeploy.ps1


---------------------------------------------------------------------------------------------------------



**Résumé Global :**

En résumé les étapes à suivre pour créer et tester le déploiement d'une image personnalisée dans Azure :
1. Télécharger et installer Terraform et Packer
1. Dans un répertoire, copier et personnaliser les fichiers .tf, le fichier Packer-XXXXX.json, ficierparametres.parameters.json, ARMDeploy.ps1
1. Lancer Terraform pour construire ressource group et créer les ressources nécesssaires (Nic, VM, Storage...) : ``Terraform apply``
1. Valider le package Packer. Exemple : ``packer validate Packer-VMRHEL73StanAzureCustom.json``
1. Builder le package Packer. Exemple: ``packer build -color=true Packer-VMRHEL73StanAzureCustom.json``
1. Télécharger le modèle ARM JSON généré par Packer (c'est un artifact dans le compte de stockage) et le renommer ex: ARM-VMRHEL73StanAzureCustom.json. Attention ce template ARM en JSON n'est généré que si le résultat du build est un vhd dans un compte de stockage. Si vous avez choisi l'option disque géré alors c'est à vous de créer / récupérer puis personnaliser un modèle ARM JSON avec la référence de l'image buildée (vous pouvez trouver des modèles sur le github Azure quickstart template)
1. Modifier le fichier parameters.json avec l'ID de la NIC créée par Terraform (visible via Terraform output) : fichierparametres.parameters.json
1. exécuter le script Powershell ``.\ARMDeploy.ps1``

En résumé les étapes à suivre pour nettoyer votre environnement :
1. Supprimer la VM créée depuis l'image générée
1. ``Terraform Destroy``
