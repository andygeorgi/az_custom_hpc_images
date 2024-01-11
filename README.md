# Build your Custom HPC Image to be used on Microsoft Azure

## Tools used

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

## Login and store subscription information

```bash
az login

# Save subscription name and ID
subscriptionName=$(az account show --query name --output tsv)
subscriptionID=$(az account show --query id --output tsv)
```

## Set environment specific variables (to be customized to own needs)

```bash
# Destination image resource group
imageResourceGroup="RGHPCImages"

# Location (see possible locations in main docs)
location="westeurope"

# VM size used to build the image (H- or N-series recommended if building with IB and/or GPU support)
vmSize="Standard_HB120rs_v2"

# Name of the Service Principal to be created
servicePrincipalName="SPpacker"

# Distribution specific configuration
# RHEL, Alma (CentOS and Ubuntu coming soon ...)
Distribution="RHEL"

# Currently RHEL 7.x or Alma 8.4 only
MajorVersion=7

# Currently:
#   * RHEL 7.x with x=6|7|8|9
#   * Alma 8.4
MinorVersion=9

# VM generation (gen1|gen2)
VMGen=gen1

# GPU support needed (true|false) - GPU support coming soon ...
GPUSupport=false

if $GPUSupport; then
        workingDirectory="${Distribution}/${Distribution}${MajorVersion}x-NvidiaGPU"
else
        workingDirectory="${Distribution}/${Distribution}${MajorVersion}x-NonGPU"
fi

packerFile="Packer-CustomHPC-${Distribution}${MajorVersion}${MinorVersion}-${VMGen}.json"
```

## Preperation steps

```bash
# Create resource group if it not already exists
az group create -n $imageResourceGroup -l $location
```

```bash
# Create Service Principal and get password
az ad sp create-for-rbac --name "http://$servicePrincipalName" --role Contributor --scopes /subscriptions/$subscriptionID --years 1 --output tsv > SPpackerCredentials.out

# Grep information about the SP
servicePrincipalPassword=$(awk '{print $3}' SPpackerCredentials.out)
servicePrincipalAppId=$(awk '{print $1}' SPpackerCredentials.out)
servicePrincipalTenant=$(awk '{print $4}' SPpackerCredentials.out)
```

## Start Packer build prcocess

```bash
packer build \
  -var "location=$location" \
  -var "vmSize=$vmSize" \
  -var "client_id=$servicePrincipalAppId" \
  -var "client_secret=$servicePrincipalPassword" \
  -var "tenant_id=$servicePrincipalTenant" \
  -var "subscription_id=$subscriptionID" \
  -var "resource_group=$imageResourceGroup" \
  ${workingDirectory}/${packerFile}
```

## Tested with

```bash
~$ cat /etc/redhat-release
Red Hat Enterprise Linux Server release 7.6 (Maipo)
Red Hat Enterprise Linux Server release 7.7 (Maipo)
Red Hat Enterprise Linux Server release 7.8 (Maipo)
Red Hat Enterprise Linux Server release 7.9 (Maipo)
```

```bash
~$ cyclecloud --version
CycleCloud 8.1.0-1275
```

```bash
~$ slurmctld -V
slurm 20.11.7
```

```bash
~$ qsub --version
pbs_version = 18.1.4
```
