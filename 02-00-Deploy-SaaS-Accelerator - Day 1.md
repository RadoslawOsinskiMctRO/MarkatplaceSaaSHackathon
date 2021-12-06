# Challenge 2: Deploy SaaS Accelerator

## Introduction
Now you will deploy management portal, which you can reuse for the purpose of your own product.

## Description
Go to aka.ms/SaaSAccelerator and read repo main readme file and follow installaction instruction.<br>
Make sure that before deployment you will register AAD Applications from sub task 02-01.<br>
Use optional parameters as well: [instalation instruction](https://github.com/Azure/Commercial-Marketplace-SaaS-Accelerator/blob/main/docs/Installation-Instructions.md)<br>

```PowerShell
git clone https://github.com/Azure/Commercial-Marketplace-SaaS-Accelerator.git -b main --depth 1; `
 cd ./Commercial-Marketplace-SaaS-Accelerator/deployment/Templates; `
 Connect-AzureAD -Confirm; .\Deploy.ps1 `
 -WebAppNamePrefix "<<your value>>" `
 -SQLServerName "<<your value>>" `
 -SQLAdminLogin "<<your value>>" `
 -SQLAdminLoginPassword "<<your value>>" `
 -PublisherAdminUsers "<<your value andmin email>>" `
 -ResourceGroupForDeployment "<<your value>>" `
 -Location "West Europe" `
 -PathToARMTemplate ".\deploy.json" `
-TenantID "XXXXXXX-XXXX-XXXX-XXXX-XXXXXXX" `
-AzureSubscriptionID "XXXXXXX-XXXX-XXXX-XXXX-XXXXXXX" `
-ADApplicationID "XXXXXXX-XXXX-XXXX-XXXX-XXXXXXX" `
-ADApplicationSecret "XXXXXXX~E_XXXXXXX5qXCjwXXXXXXX" `
-ADMTApplicationID "XXXXXXX-XXXX-XXXX-XXXX-XXXXXXX" 
```
## Success Criteria
1. You can log in to customer portal.
2. You can log in to admin portal.
3. Check if the configuration was pushed correctly in App Service settings.

## Learning Resources
aka.ms/saasaccelerator
