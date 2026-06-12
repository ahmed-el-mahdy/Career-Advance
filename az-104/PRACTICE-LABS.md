# ☁️ AZ-104 Practice Labs (Free)

> All labs use Microsoft Learn Sandboxes or Azure Free Account

---

## 🔐 Lab 1: Identity & RBAC

### Objective: Create users, assign roles, test access

```bash
# Create a resource group
az group create --name lab-rg --location eastus

# Create a user (Entra ID)
az ad user create --display-name "Lab User" --user-principal-name labuser@yourdomain.onmicrosoft.com --password "P@ssw0rd123!"

# Assign Reader role at resource group scope
az role assignment create --assignee labuser@yourdomain.onmicrosoft.com --role "Reader" --resource-group lab-rg

# Verify
az role assignment list --resource-group lab-rg --output table

# Create custom role (JSON definition)
cat <<EOF > custom-role.json
{
  "Name": "VM Operator",
  "Description": "Start and stop VMs only",
  "Actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/powerOff/action",
    "Microsoft.Compute/virtualMachines/read"
  ],
  "AssignableScopes": ["/subscriptions/<sub-id>"]
}
EOF
az role definition create --role-definition custom-role.json
```

---

## 💾 Lab 2: Storage

### Objective: Create storage, upload blobs, configure lifecycle

```bash
# Create storage account
az storage account create --name mystoragelab$(date +%s) --resource-group lab-rg --sku Standard_LRS --kind StorageV2

# Create container
az storage container create --name uploads --account-name $STORAGE_NAME

# Upload a file
echo "hello world" > test.txt
az storage blob upload --container-name uploads --file test.txt --name test.txt --account-name $STORAGE_NAME

# List blobs
az storage blob list --container-name uploads --account-name $STORAGE_NAME --output table

# Set access tier
az storage blob set-tier --container-name uploads --name test.txt --tier Cool --account-name $STORAGE_NAME

# AzCopy (practice this!)
azcopy copy "test.txt" "https://$STORAGE_NAME.blob.core.windows.net/uploads?<SAS-token>"

# Generate SAS token
az storage account generate-sas --account-name $STORAGE_NAME --permissions rwdl --expiry 2026-12-31 --resource-types sco --services b
```

---

## 🖥️ Lab 3: Virtual Machines & Scale Sets

### Objective: Deploy VMs, configure availability, autoscale

```bash
# Create VM
az vm create --resource-group lab-rg --name lab-vm --image Ubuntu2204 --admin-username azureuser --generate-ssh-keys --size Standard_B1s

# Resize VM
az vm resize --resource-group lab-rg --name lab-vm --size Standard_B2s

# Create availability set
az vm availability-set create --resource-group lab-rg --name lab-avset --platform-fault-domain-count 2

# Create VMSS
az vmss create --resource-group lab-rg --name lab-vmss --image Ubuntu2204 --upgrade-policy-mode automatic --admin-username azureuser --generate-ssh-keys --instance-count 2

# Configure autoscale
az monitor autoscale create --resource-group lab-rg --resource lab-vmss --resource-type Microsoft.Compute/virtualMachineScaleSets --min-count 2 --max-count 5 --count 2

# Add autoscale rule (CPU > 70% → scale out)
az monitor autoscale rule create --resource-group lab-rg --autoscale-name lab-vmss --condition "Percentage CPU > 70 avg 5m" --scale out 1
```

---

## 🌐 Lab 4: Networking

### Objective: VNets, peering, NSGs, Load Balancer

```bash
# Create VNet 1
az network vnet create --resource-group lab-rg --name vnet1 --address-prefix 10.1.0.0/16 --subnet-name subnet1 --subnet-prefix 10.1.1.0/24

# Create VNet 2
az network vnet create --resource-group lab-rg --name vnet2 --address-prefix 10.2.0.0/16 --subnet-name subnet1 --subnet-prefix 10.2.1.0/24

# Peer VNets
az network vnet peering create --resource-group lab-rg --name vnet1-to-vnet2 --vnet-name vnet1 --remote-vnet vnet2 --allow-vnet-access
az network vnet peering create --resource-group lab-rg --name vnet2-to-vnet1 --vnet-name vnet2 --remote-vnet vnet1 --allow-vnet-access

# Create NSG
az network nsg create --resource-group lab-rg --name web-nsg

# Add rule (allow HTTP)
az network nsg rule create --resource-group lab-rg --nsg-name web-nsg --name allow-http --priority 100 --direction Inbound --access Allow --protocol Tcp --destination-port-ranges 80

# Create Load Balancer
az network lb create --resource-group lab-rg --name lab-lb --sku Standard --frontend-ip-name frontend --backend-pool-name backend-pool

# Create health probe
az network lb probe create --resource-group lab-rg --lb-name lab-lb --name http-probe --protocol Http --port 80 --path /

# Create LB rule
az network lb rule create --resource-group lab-rg --lb-name lab-lb --name http-rule --protocol Tcp --frontend-port 80 --backend-port 80 --frontend-ip-name frontend --backend-pool-name backend-pool --probe-name http-probe
```

---

## 📊 Lab 5: Monitoring & Backup

### Objective: Azure Monitor alerts, Log Analytics, VM backup

```bash
# Create Log Analytics workspace
az monitor log-analytics workspace create --resource-group lab-rg --workspace-name lab-workspace

# Create metric alert (VM CPU > 80%)
az monitor metrics alert create --resource-group lab-rg --name high-cpu --scopes "/subscriptions/<sub>/resourceGroups/lab-rg/providers/Microsoft.Compute/virtualMachines/lab-vm" --condition "avg Percentage CPU > 80" --description "High CPU alert"

# Create Recovery Services vault
az backup vault create --resource-group lab-rg --name lab-vault --location eastus

# Enable VM backup
az backup protection enable-for-vm --resource-group lab-rg --vault-name lab-vault --vm lab-vm --policy-name DefaultPolicy

# KQL query example (run in Log Analytics)
# Heartbeat | where TimeGenerated > ago(1h) | summarize count() by Computer
# AzureActivity | where CategoryValue == "Administrative" | summarize count() by OperationNameValue
```

---

## 🧹 Cleanup

```bash
# Delete everything when done
az group delete --name lab-rg --yes --no-wait
```

---

*Use Microsoft Learn Sandboxes for free (no credit card). Use your Azure Free Account for labs that need persistence. Your Mosadad subscription gives you real-world practice daily.*
