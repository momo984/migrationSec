# Cambio Enrutamiento SAP Azure

## Login AZURE
 - LoginAzurePowerShell 

```powershell
az account clear
#az config set core.enable_broker_on_windows=false

az login
```
 - Global variable 

```powershell



$SUBCRIPTION="47156fbe-13bc-4ca6-9d1e-4008ad7f7fe7"
$SUBCRIPTIONSAP="e673010c-3eed-4126-b712-3883ffd1e870"


#getvnet id 
$ResourceGroupRoutetable="vmsles001"
$Routetablename="routetablenew"

$Routeid = az network route-table show -g $ResourceGroupRoutetable -n $Routetablename  --query id --out tsv



```


 - define variable 

```powershell


$VnetHubcompartidaid="/subscriptions/47156fbe-13bc-4ca6-9d1e-4008ad7f7fe7/resourceGroups/rg-net-shared-001/providers/Microsoft.Network/virtualNetworks/vnet-ti-shared-001"
$VnetSpoke30id="/subscriptions/e673010c-3eed-4126-b712-3883ffd1e870/resourceGroups/SAP_Servientrega/providers/Microsoft.Network/virtualNetworks/VNetSpoke-SAP-Accesos"
$Vnetspoke40id="/subscriptions/e673010c-3eed-4126-b712-3883ffd1e870/resourceGroups/SAP_Servientrega/providers/Microsoft.Network/virtualNetworks/VNetSpoke-SAP-PRD"
$Vnetspoke50id="/subscriptions/e673010c-3eed-4126-b712-3883ffd1e870/resourceGroups/SAP_Servientrega/providers/Microsoft.Network/virtualNetworks/VNetSpoke-SAP-QADEV"
$Vnetspoke60id="/subscriptions/e673010c-3eed-4126-b712-3883ffd1e870/resourceGroups/SAP_Servientrega/providers/Microsoft.Network/virtualNetworks/VNetSpoke-SAP-Pruebas"

$VnetHubcompartida="vnet-ti-shared-001"
$VnetSpoke30="VNetSpoke-SAP-Accesos"
$Vnetspoke40="VNetSpoke-SAP-PRD"
$Vnetspoke50="VNetSpoke-SAP-QADEV"
$Vnetspoke60="VNetSpoke-SAP-Pruebas"
$ResourceGroupNamespoke="SAP_Servientrega"
$ResourceGroupNamehub="rg-net-shared-001"

$Routeid="/subscriptions/e673010c-3eed-4126-b712-3883ffd1e870/resourceGroups/SAP_Servientrega/providers/Microsoft.Network/routeTables/RT_SAP_FORTIGATE"
```


 - Get RouteTable ID DONE

```powershell


#getvnet id 
$ResourceGroupRoutetable="SAP_Servientrega"
$Routetablename="RT_SAP_FORTIGATE"

$Routeid = az network route-table show -g $ResourceGroupRoutetable -n $Routetablename  --query id --out tsv

```



 - Set SUBCRIPTION 

```powershell

az account set --subscription $SUBCRIPTION
```

 - Set SUBCRIPTION SAP 

```powershell

az account set --subscription $SUBCRIPTIONSAP
```


## 1. CONFIGURAR PEERING ENTRE VNET SERVICIOS COMPARTIDOS Y VNET DE SAP Pruebas EN AZURE


### 00.	Peering 172.29.0.0/18 vs 172.31.60.0/24

 - Special for the current config 
```powershell

$PeeringNameh2spk="peer-hub-to-VNetSpoke-SAP-pruebas"
$PeeringNamespk2h="VNetSpoke-SAP-pruebas-to-peer-hub"
$Vnethub=$VnetHubcompartida
$Vnethubid=$VnetHubcompartidaid
$Vnetspoke=$VnetSpoke60
$Vnetspokeid=$VnetSpoke60id
```

 - Create peering hub to spoke
```powershell

#create peering hub to spoke
az account set --subscription $SUBCRIPTION
az network vnet peering create -g $ResourceGroupNamehub -n $PeeringNameh2spk --vnet-name $Vnethub --remote-vnet $Vnetspokeid --allow-vnet-acces 

#enable gateway from hub 
az network vnet peering update --resource-group $ResourceGroupNamehub --name $PeeringNameh2spk   --vnet-name $Vnethub  --set allow_gateway_transit=true --set allow_virtual_network_access=true

```

 - Create peering Sopke to hub 
```powershell

#create peering spoke to hub

az account set --subscription $SUBCRIPTIONSAP
az network vnet peering create -g $ResourceGroupNamespoke -n $PeeringNamespk2h --vnet-name $Vnetspoke --remote-vnet $Vnethubid --allow-vnet-acces

#enable gateway from Spokeside 
az network vnet peering update --resource-group $ResourceGroupNamespoke --name $PeeringNamespk2h  --vnet-name $Vnetspoke  --set allow_forwarded_traffic=true  --set use_remote_gateways=true --set allow_virtual_network_access=true

az account set --subscription $SUBCRIPTION

```


## Vnet Servicios compartidos: 172.29.0.0/18 & Vnet Sap : 172.31.30.0/24, 172.31.40.0/24, 172.31.50.0/24


### I.	Peering 172.29.0.0/18 vs 172.31.30.0/24

 - Special for the current config 
```powershell

$PeeringNameh2spk="peer-hub-to-VNetSpoke-SAP-Accesos"
$PeeringNamespk2h="VNetSpoke-SAP-Accesos-to-peer-hub"
$Vnethub=$VnetHubcompartida
$Vnethubid=$VnetHubcompartidaid
$Vnetspoke=$VnetSpoke30
$Vnetspokeid=$VnetSpoke30id
```

 - Create peering hub to spoke
```powershell

#create peering hub to spoke
az account set --subscription $SUBCRIPTION
az network vnet peering create -g $ResourceGroupNamehub -n $PeeringNameh2spk --vnet-name $Vnethub --remote-vnet $Vnetspokeid --allow-vnet-acces 

#enable gateway from hub 
az network vnet peering update --resource-group $ResourceGroupNamehub --name $PeeringNameh2spk   --vnet-name $Vnethub  --set allow_gateway_transit=true --set allow_virtual_network_access=true

```

 - Create peering Sopke to hub 
```powershell

#create peering spoke to hub

az account set --subscription $SUBCRIPTIONSAP
az network vnet peering create -g $ResourceGroupNamespoke -n $PeeringNamespk2h --vnet-name $Vnetspoke --remote-vnet $Vnethubid --allow-vnet-acces

#enable gateway from Spokeside 
az network vnet peering update --resource-group $ResourceGroupNamespoke --name $PeeringNamespk2h  --vnet-name $Vnetspoke  --set allow_forwarded_traffic=true  --set use_remote_gateways=true --set allow_virtual_network_access=true

az account set --subscription $SUBCRIPTION

```


### II.	Peering 172.29.0.0/18 vs 172.31.40.0/24
 
 - Special for the current config 
```powershell

$PeeringNameh2spk="peer-hub-to-VNetSpoke-SAP-PRD"
$PeeringNamespk2h="VNetSpoke-SAP-PRD-to-peer-hub"
$Vnethub=$VnetHubcompartida
$Vnethubid=$VnetHubcompartidaid
$Vnetspoke=$VnetSpoke40
$Vnetspokeid=$VnetSpoke40id
```

 - Create peering hub to spoke
```powershell

#create peering hub to spoke
az account set --subscription $SUBCRIPTION
az network vnet peering create -g $ResourceGroupNamehub -n $PeeringNameh2spk --vnet-name $Vnethub --remote-vnet $Vnetspokeid --allow-vnet-acces 

#enable gateway from hub 
az network vnet peering update --resource-group $ResourceGroupNamehub --name $PeeringNameh2spk   --vnet-name $Vnethub  --set allow_gateway_transit=true --set allow_virtual_network_access=true

```

 - Create peering Sopke to hub 
```powershell

#create peering spoke to hub

az account set --subscription $SUBCRIPTIONSAP
az network vnet peering create -g $ResourceGroupNamespoke -n $PeeringNamespk2h --vnet-name $Vnetspoke --remote-vnet $Vnethubid --allow-vnet-acces

#enable gateway from Spokeside 
az network vnet peering update --resource-group $ResourceGroupNamespoke --name $PeeringNamespk2h  --vnet-name $Vnetspoke  --set allow_forwarded_traffic=true  --set use_remote_gateways=true --set allow_virtual_network_access=true

az account set --subscription $SUBCRIPTION

```

### III.	Peering 172.29.0.0/18 vs 172.31.50.0/24

 - Special for the current config 
```powershell

$PeeringNameh2spk="peer-hub-to-VNetSpoke-SAP-QADEV"
$PeeringNamespk2h="VNetSpoke-SAP-QADEV-to-peer-hub"
$Vnethub=$VnetHubcompartida
$Vnethubid=$VnetHubcompartidaid
$Vnetspoke=$VnetSpoke40
$Vnetspokeid=$VnetSpoke40id
```

 - Create peering hub to spoke
```powershell

#create peering hub to spoke
az account set --subscription $SUBCRIPTION
az network vnet peering create -g $ResourceGroupNamehub -n $PeeringNameh2spk --vnet-name $Vnethub --remote-vnet $Vnetspokeid --allow-vnet-acces 

#enable gateway from hub 
az network vnet peering update --resource-group $ResourceGroupNamehub --name $PeeringNameh2spk   --vnet-name $Vnethub  --set allow_gateway_transit=true --set allow_virtual_network_access=true

```

 - Create peering Sopke to hub 
```powershell

#create peering spoke to hub

az account set --subscription $SUBCRIPTIONSAP
az network vnet peering create -g $ResourceGroupNamespoke -n $PeeringNamespk2h --vnet-name $Vnetspoke --remote-vnet $Vnethubid --allow-vnet-acces

#enable gateway from Spokeside 
az network vnet peering update --resource-group $ResourceGroupNamespoke --name $PeeringNamespk2h  --vnet-name $Vnetspoke  --set allow_forwarded_traffic=true  --set use_remote_gateways=true --set allow_virtual_network_access=true

az account set --subscription $SUBCRIPTION

```

## 2.	AGREGAR A LA SIGUIENTES SUBREDES A LA TABLA DE RUTAS RT_SAP_FORTIGATE 
(Ejecutor: SONDA)

 - Special for the current config 
```powershell

$VnetSpoke30="VNetSpoke-SAP-Accesos"
$Vnetspoke40="VNetSpoke-SAP-PRD"
$Vnetspoke50="VNetSpoke-SAP-QADEV"
$Vnetspoke60="VNetSpoke-SAP-Pruebas"
$ResourceGroupName="SAP_Servientrega"
$Routeid="/subscriptions/e673010c-3eed-4126-b712-3883ffd1e870/resourceGroups/SAP_Servientrega/providers/Microsoft.Network/routeTables/RT_SAP_FORTIGATE"
```


### 0.	Associate Route Table  to 172.31.60.0/24 $VnetSpoke60

 - Special for the current config 
```powershell

$vnet=$VnetSpoke60
$subnet1="SubnetDatosPRUEBAS"
$subnet2="SubnetAppPRUEBAS"

az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet1 --route-table $Routeid
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet2 --route-table $Routeid


```




### I.	Associate Route Table  to 172.31.30.0/24 $VnetSpoke30

 - Special for the current config 
```powershell

$vnet=$VnetSpoke30
$subnet1=
$subnet2=
$subnet3=
$subnet4=
$subnet5=
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet1 --route-table $Routeid
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet2 --route-table $Routeid
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet3 --route-table $Routeid
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet4 --route-table $Routeid
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet5 --route-table $Routeid

```










--------------


## Variables

 - define variable 

```powershell

$Proyecto="MiCAV"
$Ambiente="Produccion"
$ConsectivoServicio="2021122701"
$ResourceGroup="RG_MICAV_Prod"
$AKSname="CLUS-AKS-MICAVPROD"
$ACRname="crmicavprod"
$APIname="apimanagermicavprod"
apimMail=$"Jackeline.Sanchez@icbf.gov.co"
$sqlsrvrname="dbsrvmicavprod"
$sqldbname="MICAV_PROD"
$sqlusrname="UserKB-PROD"
$sqlupass="x#_ZD5Xcr4q3k5<"
$tags="Ambiente=$Ambiente Proyecto=$Proyecto ConsectivoServicio=$ConsectivoServicio"
$location="eastus2"
$SUBSCRIPTION= "a43a1ab5-f2f0-4835-9a39-8c810b7dc8b3"


################validar
$AKSversion=az aks get-versions -l $location --query 'orchestrators[-1].orchestratorVersion' -o tsv
echo $AKSversion
$AKSversion
################validar


$ApplicationGatwwayName='APG-MiCAV-Prod'


$VnetAKS='VN-MiCAV-AKS-Prod'
$VnetAKSprefix='12.2.0.0/15'

$SubnetAKS='SN-MiCAV-AKS-Prod'
$SubnetAKSprefix='12.2.0.0/16'

$appgw-subnet-CIDR='12.3.0.0/16'
$service-cidrADDress='12.11.0.0/16'
$DNSSrvsIP='12.11.0.10'
$docker-bridge-address='172.17.0.1/16'


$VnetJumpBox='VN-MiCAV-JumB-Prod'
$VnetJumpBoxprefix='12.1.0.32/27'

$SubnetJumpBox='SN-MiCAV-JumB-Prod'
$SubnetJumpBoxprefix='12.1.0.32/28'


$PeeringName='peer-VN-MiCAV-AKS-Prod-VN-MiCAV-JumB-Prod'
$PeeringNameTo='peer-VN-MiCAV-JumB-Prod-VN-MiCAV-AKS-Prod'


$VnetOnpremise='VN_VPN_GW_Onpremise'
$PeeringNameVPN='peer-VN-MiCAV-AKS-Prod-VN_VPN_GW_Onpremise'
$PeeringNameToVPN='peer-VN_VPN_GW_Onpremise-VN-MiCAV-AKS-Prod'

###########Crear Jump Box RED
NetworkSecurityGroup=$'NSG-MiCAV-JumB-PROD'
PublicIP=$'PIP-MiCAV-JumB-Prod'
NIC=$'NIC-MiCAV-JumB-Prod'
vm=$'SAZW19VIICAVJB'
admusr=$'adminJUMB'
admpassword=$'P@$$w0rdadminJUMB'



$KeyVaultName='KV-MiCAV-Prod'
$diskName='DSK-MiCAV-Prod'
#######################################################################################################################################


###########Crear Jump Box RED 

$NetworkSecurityGroup='NSG-SUIN-JumB-Prod'
$PublicIP='PIP-SUIN-JumB-Prod'
$NIC='NIC-SUIN-JumB-Prod'
$vm='SAZW19VIISUINJB'
$admusr='XXXXXXXX'
$admpassword='XXXXXXXXXXXX'


##############Crear Cluster AKS Componets
$ApplicationGatewayName='APG-SUIN-Prod'
$KeyVaultName='KV-SUIN-Prod'
$diskName='DSK-SUIN-Prod'

```





<h1 align="center">AKS PARA MiCAV Prod

  ![image](https://user-images.githubusercontent.com/62779771/142261026-45415848-1055-451e-ae1e-039b66437946.png)
  
  
  <h1 align="center">AKS PARA WSO2

![image](https://user-images.githubusercontent.com/62779771/142261026-45415848-1055-451e-ae1e-039b66437946.png)


    
  
### Crear Networking
  
  El primero paso es crear el networking definiendo los espacios de red.
```powershell  
  $ResourceGroupName = 'WU1-ICBF-POC-RGP-001'
  $Location = 'West us'
  $SUBCRIPTION = ''

  
  $VnetAKS = 'WU1-ICBF-POC-VNET-001'
  $SubnetAKS = 'WU1-ICBF-POC-SNET-001'

  $VnetJumpBox = 'WU1-ICBF-POC-VNET-002'
  $SubnetJumpBox = 'WU1-ICBF-POC-SNET-002'

  $PeeringName = 'VNET-001-VNET-002'
  $PeeringNameTo = 'VNET-002-VNET-001'

  az login
  az account set --subscription $SUBCRIPTION
  az group create --name $ResourceGroupName --location $location

  az network vnet create --address-prefixes 10.0.0.0/15 --name $VnetAKS --resource-group $ResourceGroupName --subnet-name $SubnetAKS --subnet-prefixes 10.0.0.0/16
  az network vnet create --address-prefixes 172.18.0.0/26 --name $VnetJumpBox --resource-group $ResourceGroupName --subnet-name $SubnetJumpBox --subnet-prefixes 172.18.0.0/27
  az network vnet peering create -g $ResourceGroupName -n $PeeringName --vnet-name $VnetAKS --remote-vnet $VnetJumpBox --allow-vnet-acces
  az network vnet peering create -g $ResourceGroupName -n $PeeringNameTo --vnet-name $VnetJumpBox --remote-vnet $VnetAKS --allow-vnet-acces
```
### Crear Jump Box
El AKS sera privado por lo que se creara una maquina virtual para poder acceder al cluster

```powershell  
  $NetworkSecurityGroup = 'WU1-ICBF-POC-NSG-001'
  $PublicIP = 'WU1-ICBF-POC-PIP-001'
  $NIC = 'WU1-ICBF-POC-NIC-001'
  $vm = 'ICBFPOCJUMPBOX'

  az network nsg create --resource-group $ResourceGroupName --name $NetworkSecurityGroup
  az network nsg rule create --resource-group $ResourceGroupName --nsg-name $NetworkSecurityGroup --name Allow-SSH-Internet --access Allow --protocol Tcp --direction Inbound --priority 100 --source-address-prefix Internet --source-port-range "*" --destination-address-prefix "*" --destination-port-range 3389
  az network public-ip create --name $PublicIP --resource-group $ResourceGroupName
  az network nic create --resource-group $ResourceGroupName --name $NIC --vnet-name $VnetJumpBox --subnet $SubnetJumpBox --accelerated-networking true --public-ip-address $PublicIP --network-security-group $NetworkSecurityGroup
  az vm create --resource-group $ResourceGroupName --name $vm --image Win2019Datacenter --size Standard_DS4_v2 --admin-username <user> --admin-password <password> --nics $NIC
```
### Crear RBAC VNET 
Se debe de copiar AppId y Password se utilizara en el siguiente paso
```powershell  
az ad sp create-for-rbac --skip-assignment
```
Copiar AppId y Password

### Crear Cluster AKS Componets

  Ejecutar los siguientes scripts para crear el cluster de AKS habilitando el application Gateway y los porviders para key vault

```powershell
 $ClusterName = 'WU1-ICBF-POC-AKS-001'
  $ApplicationGatwwayName ='WU1-ICBF-POC-AG-001'
  $KeyVaultName = 'WU1-ICBF-POC-KVR-001'
  $diskName = 'WU1-ICBF-POC-DSK-001'
  $ACR_NAME = 'WU1ICBFPOCACR001'

  
  $VNET_ID=$(az network vnet show --resource-group $ResourceGroupName --name $VnetAKS --query id -o tsv)
  $SUBNET_ID=$(az network vnet subnet show --resource-group $ResourceGroupName --vnet-name $VnetAKS --name $SubnetAKS --query id -o tsv)
  az role assignment create --assignee "<applicationId>" --scope $VNET_ID --role Contributor
  
  az aks create -g $ResourceGroupName -n $ClusterName -a ingress-appgw --appgw-name $ApplicationGatwwayName --appgw-subnet-cidr "10.1.0.0/16" --location $Location --network-plugin azure --vnet-subnet-id $SUBNET_ID --service-cidr "10.3.0.0/16" --dns-service-ip "10.3.0.10" --docker-bridge-address 172.17.0.1/16  --service-principal "<applicationId>" --client-secret "<password>" --enable-managed-identity --enable-private-cluster --generate-ssh-keys

  az aks enable-addons --addons azure-keyvault-secrets-provider --name $ClusterName --resource-group $ResourceGroupName
```

- Crear Key Vault y Secrets
```powershell
az keyvault create -n $KeyVaultName -g $ResourceGroupName -l $Location
az keyvault secret set --vault-name $KeyVaultName -n UserNameBonita --value <valor>
az keyvault secret set --vault-name $KeyVaultName -n PasswordBonita --value <valor>
```
- Crear Identidad
```powershell
az identity create -g $ResourceGroupName -n identityAKS 
```
  
Copiar identidad ID generado (identityID) y el Client ID  (clientID)

- Asignar Identidad to scale set
Tomar los valores de la maquina virtual autogenerada
   
```powershell
 
  az vmss identity assign -g "<rg>" -n "<vmss>" --identities <identityID>
```
  
- Asignar accesos de identidad al keyvault
  
```powershell
   $ClientId = <"clientID">
  az keyvault set-policy -n $KeyVaultName --key-permissions get --spn $ClientId
  az keyvault set-policy -n $KeyVaultName --secret-permissions get --spn $ClientId
  az keyvault set-policy -n $KeyVaultName --certificate-permissions get --spn $ClientId
```
- Crear ACR y DISK para AKS

```powershell
  az acr create --resource-group $ResourceGroupName --location $location --name $ACR_NAME --sku Standard
  az disk create --resource-group $ResourceGroupName --name $diskName --size-gb 1 --query id --output tsv
  az aks update -n $ClusterName -g $ResourceGroupName --attach-acr $ACR_NAME
```
### Instalación JUMP BOX

- Entrara a la VM mediante RDP con las credenciales configuradas.
- Instalar la ultima version de Azure CLI https://aka.ms/installazurecliwindows
- Instalar Kubectl en powershell
 
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
choco install kubernetes-cli -y
kubectl version --client
```
- Agregar VNET en Private DNS Zone
<img src="https://user-images.githubusercontent.com/62779771/142080359-f3b801e0-eecb-442d-8f2e-68319294b247.png" data-canonical-src="https://user-images.githubusercontent.com/62779771/142080359-f3b801e0-eecb-442d-8f2e-68319294b247.png" width="200" height="200" align="center"/>
  
### Instalacion Componentes AKS  
  
- Verificar instalación de providers para keyvault

```powershell
az login
$SUBCRIPTION = ''
az account set --subscription $SUBCRIPTION

$ResourceGroupName = 'WU1-ICBF-POC-RGP-001'
$ClusterName = 'WU1-ICBF-POC-AKS-001'
az aks get-credentials --resource-group $ResourceGroupName --name $ClusterName

kubectl get pods -n kube-system -l 'app in (secrets-store-csi-driver, secrets-store-provider-azure)'
```  
- Descargar Archivos
  
  [azureprovider](https://github.com/RodrigoVeraSYS/AKS-Steps/blob/main/azureprovider.yaml "azureprovider")
  
  [validatekeyvault](https://github.com/RodrigoVeraSYS/AKS-Steps/blob/main/validatekeyvault.yaml "validatekeyvault")
  
- Crear SecretProviderClass object
  Se debe de ejecurar el YAML azureprovider.yaml sustituyendo el nombre del Key Vault, Resource Name, SubscriptionId y Tenantid
```powershell  
  kubectl apply -f  .\azureprovider.yaml
  kubectl apply -f  .\validateKeyVault.yaml
  
  
  kubectl exec busybox-secrets-store-inline-user-msi -- ls /mnt/secrets-store/
  kubectl exec busybox-secrets-store-inline-user-msi -- cat /mnt/secrets-store/UserNameBonita
  kubectl exec busybox-secrets-store-inline-user-msi -- cat /mnt/secrets-store/PasswordBonita
```
### Instalación aplicacion de prueba
- Descargar Archivo
  [appprueba](https://github.com/RodrigoVeraSYS/AKS-Steps/blob/main/sampleaspnet1.yaml "appprueba")

  
  Ejecutar comando para instalar la aplicacion
```powershell  
  kubectl apply -f  .\sampleaspnet1.yaml
```
  Desde el portal de Azure obtener la IP pública del application gateway y ejecutarla desde un explorador. 
  Se debe de abrir una aplicación de prueba de ASP NET

  ![image](https://user-images.githubusercontent.com/62779771/142260290-220a3775-2c53-4229-b011-b004a2780fcd.png)
  
  <h1 align="center">AKS BONITA
  
https://hub.docker.com/_/bonita/
  
### Eliminar Datos Prueba
 
- Eliminar Disk desde el portal
- Eliminar azure secrets
```powershell  
$ResourceGroupName = 'WU1-ICBF-POC-RGP-001'
$Location = 'West us'
$SUBCRIPTION = ''
$ClusterName = 'WU1-ICBF-POC-AKS--001'
$ApplicationGatwwayName ='WU1-ICBF-POC-AG-001'
$KeyVaultName = 'WU1-ICBF-POC-KVR-003'
$diskName = 'WU1-ICBF-POC-DSK-001'
$RGAlterno = 'MC_WU1-ICBF-POC-RGP-001_WU1-ICBF-POC-AKS-001_westus'

az login
az account set --subscription $SUBCRIPTION



az keyvault secret delete --vault-name $KeyVaultName -n UserNameBonita
az keyvault secret delete --vault-name $KeyVaultName -n PasswordBonira
```


### Crear Keys Bonita
```powershell  
az keyvault secret set --vault-name $KeyVaultName -n postgres-password --value ''
az keyvault secret set --vault-name $KeyVaultName -n tenant-login --value ''
az keyvault secret set --vault-name $KeyVaultName -n tenant-password --value ''
az keyvault secret set --vault-name $KeyVaultName -n platform-login --value ''
az keyvault secret set --vault-name $KeyVaultName -n platform-password --value ''
```

### Crear Disk Bonita
```powershell  
az disk create --resource-group $ResourceGroupNameCreado --name $diskName --size-gb 1 --query id --output tsv
```
### AKS Services Docs

- ClusterIP
An internal IP address for use within the Kubernetes cluster. Used for internal communications between workloads within the cluster.  This is the default service type when none is specified.
<img src="https://user-images.githubusercontent.com/62779771/144475585-11efde64-06be-4e44-8310-51e5e2dc7787.png" data-canonical-src="https://user-images.githubusercontent.com/62779771/144475585-11efde64-06be-4e44-8310-51e5e2dc7787.png" width="200" height="200" align="center" />

- NodePort 
Creates a port mapping to the underlying node, which allows external access directly to Pods by using the Node IP address (any node’s IP) and the Node Port.
<img src="https://user-images.githubusercontent.com/62779771/144475389-5437aaca-2d52-4648-a146-ae732ea4b91c.png" data-canonical-src="https://user-images.githubusercontent.com/62779771/144475389-5437aaca-2d52-4648-a146-ae732ea4b91c.png" width="200" height="200" align="center" />

- LoadBalancer 
Requests an external IP address (public by default) from the Azure Load Balancer.  Azure creates an IP Address resource and connects the selected Pods to the load balancer backend pool.  Allow external traffic to reach Pods in the cluster.
<img src="https://user-images.githubusercontent.com/62779771/144475470-57c0b80e-9971-421c-9b67-57b394fde752.png" data-canonical-src="https://user-images.githubusercontent.com/62779771/144475470-57c0b80e-9971-421c-9b67-57b394fde752.png" width="200" height="200" align="center" />

### Kubectl Examples
```powershell  
Example: Get a list of Nodes
		kubectl get nodes
Example: Retrieve the YAML definition of a Pod from the Kubernetes database
		kubectl get pod pod-123 –o yaml
Example: Show additional details of a service
		kubectl describe service/myservice
Example: Load the object(s) specified in the yaml file into the Kubernetes database
		kubectl apply –f .\myfile.yaml
Example: View the logs from a container in a Pod
		kubectl logs pod-123 –c nginx
Example: Execute a command in a container in a Pod
		kubectl exec –it pod-123 –c nginx –- bash
```
### Kubectl Deployments
- Rolling Update - Consists of slowly rolling out a new version of an application by replacing instances one after the old version until all the instances are rolled out.  This is the default strategy for Kubernetes Deployments when none is specified.
- Recreate - Consists of shutting down all instances of the current version, then deploying the new version. This technique implies downtime of the service that depends on both shutdown and boot duration of the application.  
- Blue/Green – Consists of deploying the new version (green) alongside the current version (blue) with the same number of instances. After testing that the new version meets all the requirements, the traffic is switched from old to new version at the load balancer.  This requires using two Kubernetes Deployments and at least one Kubernetes Service.
- Canary – Consists of gradually shifting production traffic from old to new version. Usually, the traffic is split based on weight.  While this strategy can be “simulated” using a Kubernetes Service, it’s often implemented using a 3rd party Ingress Controller.
  
### Namespaces
Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces. 
Namespaces are intended for use in environments with many users spread across multiple teams, or projects.
Namespaces provide a scope for names. Names of resources need to be unique within a namespace, but not across namespaces.


### Create Namespace Bonita
```powershell
kubectl create namespace bonita
```
### Create SecretProviderClass
- Aplicar YAML
```powershell
kubectl apply -f azureprovider.yaml -n bonita
```
### Create Postgres Bonita
- Aplicar YAML
```powershell
kubectl apply -f postgress-pvc-dep-svc.yaml -n bonita
```
- Verificar que los nodos se crearon satisfactoriamente y esten en running
```powershell
kubectl get pods -n bonita
kubectl describe pods -n bonita <nombrenodo>
```

### Create Web Bonita
- Aplicar YAML
```powershell
kubectl apply -f Bonita-dep-svc-pvc.yaml -n bonita
```
- Verificar que los nodos se crearon satisfactoriamente y esten en running
```powershell
kubectl get pods -n bonita
kubectl describe pods -n bonita <nombrenodo>
```
### Obtener IPs Servicio Bonita
```powershell
$ kubectl get nodes -o wide
```
  
### Pasos Recuperacion Cluster
- Crear nuevo cluster uilizando la guia de este documento
- Mover el disco del cluster A hacia el cluster B
- Modificar YAML Bonita-dep-svc-pvc.yaml y postgress-pvc-dep-svc.yaml
Se debera obtener el nuevo ID del disco en el nuevo resource group (Resource group creado automatico)
```powershell
 volumes:
       - name: bonita-persistent-storage
         azureDisk:
          kind: Managed
          diskName: WU1-ICBF-POC-DSK-001
          diskURI: /subscriptions/3ed2fc1b-46e2-4aa1-a951-d0b09fb60bcd/resourceGroups/MC_WU1-ICBF-POC-RGP-001_WU1-ICBF-POC-AKS-001_westus/providers/Microsoft.Compute/disks/WU1-ICBF-POC-DSK-001
```
- Aplicar los YAMLS en el orden especificado con el cambio del ID de disco.


  
  
