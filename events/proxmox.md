# Proxmox

## [Documentation](https://pve.proxmox.com/pve-docs/pve-admin-guide.html)
- Login
  - Username: `root`
  - Password: `password`

## [How to get an Azure VM with ProxMox](https://chatgpt.com/share/68627b5b-7078-8005-9a54-516293f966d1)
  ```
  az storage account create -n saproxmox -g rg-proxmox -l eastus2 --sku Standard_LRS
  az storage container create --account-name saproxmox --name vhds
  az storage blob upload --account-name saproxmox --container-name vhds --file "C:\Users\mbuch\OneDrive\Desktop\proxmox\proxmox.vhd" --name proxmox.vhd

  az disk create --resource-group rg-proxmox --name disk-proxmox --source https://saproxmox.blob.core.windows.net/vhds/proxmox.vhd --os-type Linux
  ```
