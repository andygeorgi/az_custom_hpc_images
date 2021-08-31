# Custom HPC image for Azure based on RHEL 7.7

## Tools
```bash
~$ az version
    {
      "azure-cli": "2.27.1",
      "azure-cli-core": "2.27.1",
      "azure-cli-telemetry": "1.0.6",
      "extensions": {
        "image-copy-extension": "0.2.8",
        "ssh": "0.1.5"
      }
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

## Login and set environment specific variables

```bash
az login
```
```bash
# destination image resource group
imageResourceGroup="RGHPCImages"

# location (see possible locations in main docs)
location="westeurope"

# your subscription name and ID
subscriptionName=$(az account show --query id --output tsv)
subscriptionID=$(az account show --query id --output tsv)

# Name of the Service Principal to be created
servicePrincipalName="SPpacker"

#path for HPC scripts (see https://github.com/Azure/azhpc-images to get the right path for the selected image)
HPCscripts="centos/centos-7.x/centos-7.7-hpc/"
```

## Preperation steps

```bash
# create resource group if it not already exists
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
