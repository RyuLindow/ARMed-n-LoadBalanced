## Step1: Prep before resource creation
```powershell
az account show

az account set --name "INPUT YOUR TENANT NAME"
```

## Step 2: Define tags

>**Note:** Hardcoded variables. Edit for each case

```powershell
$uniqueName="umbrazing"
$RGname="LBv13"
$tagValue="loadbalanced"
```

>**Note:** You can leave this as is or change if needed.
```powershell
$location="West Europe"
$ASPtier="Basic"
$DBtier="Basic"
$DotNetVersion = "v8.0"
$username = "lindow"
$password = "pa55word"
$YourIp = "Your IP range"
$YourIpStart = "123.456.78.108"
$YourIpEnd = "123.456.78.108"
```

>**Note:** You can leave this as is or change if needed.
- [ASP tiers](https://azure.microsoft.com/en-us/pricing/details/app-service/windows/)
- [DB tiers](https://learn.microsoft.com/en-us/powershell/module/az.sql/new-azsqldatabase?view=azps-8.3.0#-edition)

```powershell
$tag=@{ "tag1"=$tagValue; "tag2"=$uniqueName }
$ASPname="asp-"+$RGname.ToLower()
$ServerName=$RGname.ToLower()+"-serv"
$webAppName=$RGname+"-"+$uniqueName
$DBname="db-"+$webAppName

$User = $username
$Pass= ConvertTo-SecureString -String $password -AsPlainText -Force
$Creds= New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User, $Pass

$DbConnString = @{umbracoDbDSN = @{Type="SQLAzure";Value="Server=$ServerName.database.windows.net,1433;database=$DBname;User ID=$username;Password=$password;Connection Timeout=300;"}}
```

## RG creation
```powershell
New-AzResourceGroup  -location $location -tag $tag -name $RGname
```
## ASP creation
**If you want a Linux OS then you need to add ```-Linux``` to the New-AzAppServicePlan command**
```powershell
New-AzAppServicePlan -location $location -tag $tag -ResourceGroupName $RGname -tier $ASPtier -name $ASPname
```
## Web app creation
```powershell
New-AzWebApp -location $location -ResourceGroupName $RGname -AppServicePlan $ASPname -name $webAppName
```
**Sets the dotnetframeworkversion**

**Setting the runtime stack doesn't work yet**
```
Set-AzWebApp -ResourceGroupName $RGname -name $webAppName $RuntimeStack -NetFrameworkVersion $DotNetVersion
```
**Adds the connection string**
```
Set-AzWebApp -ResourceGroupName $RGname -name $webAppName -ConnectionStrings $DbConnString
```
## SQL server creation
>**Note:** You'll be asked to create a user and a password, so make sure to save them.

```powershell
New-AzSqlServer -location $location -tag $tag -ResourceGroupName $RGname -ServerName $ServerName -ServerVersion "12.0" -SqlAdministratorCredentials $Creds
```
## SQL DB creation
```powershell
New-AzSqlDatabase -tag $tag -ResourceGroupName $RGname -ServerName $ServerName -Edition $DBtier -DatabaseName $DBname
```

# Copy paste
```powershell
$uniqueName="umbrazing"
$RGname="LBv13"
$tagValue="loadbalanced"
$location="West Europe"
$ASPtier="Basic"
$DBtier="Basic"
$DotNetVersion = "v8.0"
$username = "lindow"
$password = "g99xzmKoUd63T5vxDCgkN172SnlKO." # You need a very complex password or the database creation will fail with an error.
$YourIp = "Your IP range"
$YourIpStart = "123.456.78.108"
$YourIpEnd = "123.456.78.108"

$tag=@{ "tag1"=$tagValue; "tag2"=$uniqueName }
$ASPname="asp-"+$RGname.ToLower()
$ServerName=$RGname.ToLower()+"-serv"
$webAppName=$RGname+"-"+$uniqueName
$DBname="db-"+$webAppName

$User = $username
$Pass= ConvertTo-SecureString -String $password -AsPlainText -Force
$Creds= New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User, $Pass

$DbConnString = @{umbracoDbDSN = @{Type="SQLAzure";Value="Server=$ServerName.database.windows.net,1433;database=$DBname;User ID=$username;Password=$password;Connection Timeout=300;"}}

New-AzResourceGroup  -location $location -tag $tag -name $RGname

New-AzAppServicePlan -location $location -tag $tag -ResourceGroupName $RGname -tier $ASPtier -name $ASPname

New-AzWebApp -location $location -ResourceGroupName $RGname -AppServicePlan $ASPname -name $webAppName

Set-AzWebApp -ResourceGroupName $RGname -name $webAppName -NetFrameworkVersion $DotNetVersion

Set-AzWebApp -ResourceGroupName $RGname -name $webAppName -ConnectionStrings $DbConnString

New-AzSqlServer -location $location -tag $tag -ResourceGroupName $RGname -ServerName $ServerName -ServerVersion "12.0" -SqlAdministratorCredentials $Creds

New-AzSqlServerFirewallRule -ResourceGroupName $RGname -ServerName $ServerName -FirewallRuleName $YourIp -StartIpAddress $YourIpStart -EndIpAddress $YourIpEnd

New-AzSqlDatabase -tag $tag -ResourceGroupName $RGname -ServerName $ServerName -Edition $DBtier -DatabaseName $DBname
```