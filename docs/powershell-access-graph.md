# Use Azure PowerShell to access Microsoft Graph API

## Introduction

- Microsoft has been consolidating various APIs under one common namespace (REST endpoint): `graph.microsoft.com`
- Access to this unified endpoint is protected by oAuth authentication

## Application identity

The first thing that we need to do is to tell Azure Active Directory that we’re writing an application (or rather a PoSH script) that will access the graph. We’ll then use this application definition to align the permissions that tell Azure what information the script is allowed to access.

Step by step:

1. Create a new Application registration in the Azure Portal, AAD blade; choose the Web app /API Application type and provide a made up logon URL.

   - Create an Application object in AAD using `New-AzureRmADApplication`.
   - Once created the Application object in AAD, and obtained its ApplicationID, you need to create a Service Principal object using  `New-AzureRmADServicePrincipal` cmdlet, associated to the ApplicationID.
   - It is important to understand the [difference](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-application-objects) between Application and Service Principal. You can consider the application object as the global representation of your application for use across all tenants, and the service principal as the local representation for use in a specific tenant.

2. Open the new application settings, select Keys. Add a description and set the duration to 2 Years. Save that key together with the Application ID and Tenant ID to your "notepad".
3. Add required permissions to APIs - Required Permissions blade; Add - Microsoft Graph; select needed permissions (e.g. select “Read all Groups” and “Read all users’ full profiles”).
4. It’s important to understand that at this point, you have only requested the permissions that the app needs. If this were a client app, the User would have to consent to these permissions before you could do anything. In this case, we’re using App only permissions, so **the Admin must consent** by clicking on Grant Permissions link. _Note: you must consent again, if you change the list of required permissions, and you must do it separately for each API._
5. Get the Endpoint URL that we will use to obtain the Authorisation Token - value from the OAUTH 2.0 TOKEN ENDPOINT which should start https://login.windows.net.

## Example PowerShell script

```powershell
##Tenant and App Specific Values
##Add the ones that you captured during the Azure portal piece here!
$appID = "<< replace with your application ID >>"
$appSecret="<< replace with your application secret>>"

$tokenAuthURI = "<< Replace with your OAUTH 2.0 TOKEN ENDPOINT >>"

##We create a small text body with the values
$requestBody = "grant_type=client_credentials" +
    "&client_id=$appID" +
    "&client_secret=$appSecret" +
    "&resource=https://graph.microsoft.com/"

##Then we use the Token Endpoint URI and pass it the values in the body of the request
$tokenResponse = Invoke-RestMethod -Method Post -Uri $tokenAuthURI -body $requestBody -ContentType "application/x-www-form-urlencoded"

##This response provides our Bearer Token
$accessToken = $tokenResponse.access_token

##We set up which Graph endpoint we want to call (See https://graph.microsoft.io for more!)
$groupsListURI = "https://graph.microsoft.com/v1.0/groups"

##Then we make an authenticated request to the Graph API specified using the Bearer Token in the authorisation header.
$graphResponse = Invoke-RestMethod -Method Get -Uri $groupsListURI -Headers @{"Authorization"="Bearer $accessToken"}

#And then just walked through the nicely formatted PowerShell object it returns.
foreach ($group in $graphResponse.value)
{
    write-host $group.displayname
}
```

## Other way of retrieving a Bearer token

```powershell

# Get needed values
$mysecurepassword = ConvertTo-SecureString $myappPassword -AsPlainText -Force
$myusername = "" + $myapp1.ApplicationId + "@" + $ADdomainName + ""
Try {
    $mycredential = New-Object -TypeName pscredential -ArgumentList $myusername, $mysecurepassword
    Write-Host ("AAD Credential Created...") -ForegroundColor Yellow
}
Catch {
    echo 'Error creating AAD Credential!'
}
$tenant = (Get-AzSubscriptuion -SubscriptionName $mySubName).TenantId
Login-AzAccount -Credential $mycredential -ServicePrincipal -TenantId $tenant -Subscriptionname $mySubName

# Obtaion a valid Bearer token from AAD
$tenantid = $tenant
$SubscriptionId = $mySubID
$ApplicationID = $myapp1.ApplicationId
$ApplicationKey = '<key>'
$TokenEndpoint = {https://login.windows.net/{0}/oauth2/token} -f $tenantid
$ARMResource = "https://management.azure.com";

$Body = @{
    'resource' = $ARMResource
    'client_id' = $ApplicationID
    'grant_type' = 'client_credentials'
    'client_secret' = $ApplicationKey
}

$params = @{
    ContentType = 'application/x-www-form-urlencoded'
    Headers = @{'accept'='application/json'}
    Body = $Body
    Method = 'Post'
    URI = $TokenEndpoint
}

$token = Invoke-RestMethod @params
$token | select *
@{L='Expires';E={[timezone]::CurrentTimeZone.ToLocalTime(([datetime]'1/1/1970').AddSeconds($_.expires_on))}} | fl *

# Execute REST calls
$baseURI = "https://management.azure.com"
$suffixURI = "?api-version=2016-09-01"
$SubscriptionURI = $baseURI + "/subscriptions/$SubscriptionID" + $suffixURI
$uri = $SubscriptionURI
$params = @{
    ContentType = 'application/x-www-form-urlencoded'
    Headers = @{
        'authorization' = "Bearer $($Token.access_token)"
    }
    Method = 'Get'
    URI = $uri
}

$response = Invoke-RestMethod @params
$response | convertto-json # Result is not JSON formatted by default

# Example for creating a storage account
$suffixURI = "?api-version=2016-12-01"
$baseURI = "https://management.azure.com"
$uri = $baseURI + ((Get-AzResourceGroup -Name $rgname).ResourceId) + "/providers/Microsoft.Storage/storageAccounts/" + "$storageaccountname" + $suffixURI

$BodyString = "{ `
    'sku': {
        'name': '" + $storagetype + "' `
        }, `
    'location': '" + $location + "' `
    }"

$params = @{
    ContentType = 'application/json'
    Headers = @{
        'authorization'="Bearer $($Token.access_token)"
    }
    Body = $BodyString
    Method = 'Put'
    URI = $uri
}
$response2 = Invoke-WebRequest @params

```

More examples on [GitHub](https://raw.githubusercontent.com/igorpag/PSasRESTclientAzure/master/PowerShellClientForREST%20(V3c).ps1)
