# Custom HPC image for Azure based on RHEL 7

## Tested with ...

```bash
~$ cat /etc/redhat-release
Red Hat Enterprise Linux Server release 7.7 (Maipo)
```

```bash
~$ cyclecloud --version
CycleCloud 8.1.0-1275
```

## Tools
```bash
~$ az version
    {
      "azure-cli": "2.27.1",
      "azure-cli-core": "2.27.1",
      "azure-cli-telemetry": "1.0.6"
    }
```
```bash
~$ packer -v
    1.7.4
```
```bash
~$ bash --version
GNU bash, version 4.4.20(1)-release (x86_64-pc-linux-gnu)
Copyright (C) 2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html
```

## Login and set environment specific variables (to be customized to own needs)

```bash
az login
```
```bash
# Destination image resource group
imageResourceGroup="RGHPCImages"

# Location (see possible locations in main docs)
location="westeurope"

# Your subscription name and ID
subscriptionName=$(az account show --query id --output tsv)
subscriptionID=$(az account show --query id --output tsv)

# Name of the Service Principal to be created
servicePrincipalName="SPpacker"

# Specify RHEL 7 minor version to be used (6|7|8|9)
RHEL7MinorVersion=7

# Path for HPC scripts (see https://github.com/Azure/azhpc-images to get the right path for the selected image)
HPCscripts="centos/centos-7.x/centos-7.${RHEL7MinorVersion}-hpc/"
```

## Preperation steps

```bash
# Create resource group if it not already exists
az group create -n $imageResourceGroup -l $location
```

```bash
# Create Service Principal and get password
az ad sp create-for-rbac --name "http://$servicePrincipalName" --role Contributor --years 1 --output tsv > SPpackerCredentials.out

# Grep information about the SP
servicePrincipalPassword=$(awk '{print $4}' SPpackerCredentials.out)
servicePrincipalAppId=$(awk '{print $1}' SPpackerCredentials.out)
servicePrincipalTenant=$(awk '{print $5}' SPpackerCredentials.out)
```

## Start Packer build prcocess

```bash
packer build \
  -var "client_id=$servicePrincipalAppId" \
  -var "client_secret=$servicePrincipalPassword" \
  -var "tenant_id=$servicePrincipalTenant" \
  -var "subscription_id=$subscriptionID" \
  -var "resource_group=$imageResourceGroup" \
  -var "HPCscripts=$HPCscripts" \
  -var "RHEL7MinorVersion=$RHEL7MinorVersion" \
  Packer-CustomHPC-RHEL7.json
```
