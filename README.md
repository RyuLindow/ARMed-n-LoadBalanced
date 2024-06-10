# ARMed n loadbalanced: IaC for a loadbalanced setup in Azure.

This is one of the fun projects I worked on over a year ago. Although I wanted to write this article last year, the usual thing happened: the fun project got put on hold...

Now that we're here, I want to share some Infrastructure as Code (IaC) for creating a tested and approved load-balanced setup of Umbraco in Azure.

Here's a diagram showing how we recommended the setup at Umbraco HQ during the Loadbalancing training when I worked there and it's still the same AFAIK:

![Full diagram](/pics/LB%20Diagram.png)

This diagram includes some do's and don'ts notes I've added after testing this across several different issues we've encountered with the setup.

## Minimal version

Here’s a diagram of the required services for a load-balanced setup. The additional services you see in the first diagram, such as Redis cache, Storage account/container, and App Insights, are not required for this setup to work.

![Simple diagram](/pics/loadbalanced-infrastructure.png)


# [IaS PS script with Explanation](/Create-Azure-resources-using-PowerShell.md)

I initially wrote the IaC script in PowerShell for reasons I can't remember. While I recommend using ARM templates, I’ve included the PowerShell script here to help you understand the solution better.

The PS script has some notes and the script split into command, as well, as the entire block you can copy paste into PowerShell at once.

# IaC Arm

This is the actual IaC ARM template defining the services you want to create. Feel free to change the service tiers to fit your needs.

There are two ways to do it:

## [Template](/azuredeploy.json)
The actual IaC ARM template defining the services you want to created. Feel free to change the service tiers to fit your needs.

## [Parameters](/azuredeploy.parameters.json)
This is the file I recommend you edit to set the parameter values you want.

## 1. Upload the files directly to Azure.

1. Go to Azure and create a resource group for the test
2. Go to [Custom template deployment view](https://portal.azure.com/#create/Microsoft.Template)
3. Choose the **Build your own template in the editor** option and add both the **azuredeploy.json** and **azuredeploy.parameters.json** files by clicking on the **Load files** icon. Load the template file first, then the parameters file.

 You'll get a warning in at the beginning of the file on the schema line but ignore it.[It's an UI error Microsoft chose to not fix...](https://github-com.translate.goog/Azure/azure-resource-manager-schemas/issues/834?_x_tr_sl=en&_x_tr_tl=da&_x_tr_hl=da&_x_tr_pto=sc#issuecomment-585957493)

It should look like this after you add both files. You can manually edit some of the details and select a different resource group here.
![alt text](/pics/screenshot.png)

5. Click on Create and wait for the deployment to finish. It should take 3~ minutes.


## 2. Deploy the files directly from PowerShell

1. The following to connect to your tenant and select the subscription you want to create the services in.

```powershell
Connect-AzAccount 

Get-AzSubscription

$context = Get-AzSubscription -SubscriptionId {}
Set-AzContext $context
```

2. Make sure to update the first 4 variables and then run the script from below.

```powershell
$templateFile = "azuredeploy.json"
$parameterFile = "azuredeploy.parameters.json"
$RGname = "ARMed-n-LoadBalanced"
$location = "East US"
$guid = New-Guid
$random = Get-Random -Maximum 1995
$deploymentName = "deployment-" + "$guid"
New-AzResourceGroup -Name $RGname -Location $location
New-AzResourceGroupDeployment -Name $deploymentName -TemplateFile $templateFile -TemplateParameterFile $parameterFile -ResourceGroupName $RGname 


-WhatIf -WhatIfResultFormat FullResourcePayloads # By appending these two flags at the end of the script, you'll execute a dry run with logs. It's useful to see the results of the deployment without actually creating any resources.
```

All of above was tested used to find several issues with loadbalancing, resolving, and testing issues afterward. We've used it to create load balanced setups in a span of minutes and avoiding having to do ClickOps in Azure, as it's not very time efficient and prone to human errors. 

The most important take learning from all of our trouble was that the separate app service plans are necessary. The Publisher cannot operate efficiently on a scaled-out app service plan due to potential issues like cache inconsistency, file locking, and concurrency problems. Given that load balancing entails scaling out the Subscriber, each must have its own app service plan. Moreover, the Publisher plan should be a single-instance plan to ensure optimal performance.

[...as well as this section in the official docs ](https://docs.umbraco.com/umbraco-cms/fundamentals/setup/server-setup/load-balancing/azure-web-apps#azure-requirements)


I hope this gives you a clear idea of the Azure infrastructure for a load-balanced Umbraco setup. Remember, **NEVER** scale out(several instances) the publisher, only the subscribers. You're free to scale up the publisher if needed(more compute power) Otherwise, you'll end up with the subscribers not syncing "instantly."