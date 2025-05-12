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
$Vnetspoke=$VnetSpoke50
$Vnetspokeid=$VnetSpoke50id
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
$subnet1="SubnetRouter"
$subnet2="SubnetFiori"
$subnet3="SubnetSolman"
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet1 --route-table $Routeid
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet2 --route-table $Routeid
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet3 --route-table $Routeid

```

### II.	Associate Route Table  to 172.31.40.0/24 $VnetSpoke40

 - Special for the current config 
```powershell

$vnet=$VnetSpoke40
$subnet1="SubnetAppPRD"
$subnet2="SubnetDatosPRD"

az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet1 --route-table $Routeid
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet2 --route-table $Routeid
```
### III.	Associate Route Table  to 172.31.50.0/24 $VnetSpoke50

 - Special for the current config 
```powershell

$vnet=$VnetSpoke50
$subnet1="SubnetAppQAS"
$subnet2="SubnetDatosQAS"
$subnet3="SubnetAppDEV"
$subnet4="SubnetDatosDEV"
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet1 --route-table $Routeid
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet2 --route-table $Routeid
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet3 --route-table $Routeid
az network vnet subnet update -g $ResourceGroupName --vnet-name $Vnet -n $subnet4 --route-table $Routeid

```









--------------

  
  
