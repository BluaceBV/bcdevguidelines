# Bluace development guidelines for Business Central

## BC Container Helper
Microsoft created a PowerShell module that helps us with automating recurring development tasks based on docker containers, BC Container Helper. Besides that it's used as part of build pipelines (Azure DevOps) and Actions (GitHub), we also use BC Container Helper for creating our decentralized development environments.

### Creating local development environment
In general, what we need to do is determine:
- A name for the container that must be created.
- A username and password that will be used to enter Business Central in the container.
- The location where the .bclicense file can be found. The latest version is always published on [Microsoft Teams](https://bluace.sharepoint.com/:f:/s/Bluace-Products/Euv5p_uVj5ZIu5oGXCzhBesB4xuzbaX25kdXHHPpAwcUCQ?e=WIOxnS).
- Which Business Central version and localization must be used.
- Whether testing apps apps are necessary to be included into the container.
- How many memory the container is allowed to use.
- Which apps are dependent in the specific app situation, those must be installed by the the script in order to be able to publish the extension at hand into the container.

The system will then create a container for you with the desired name, which you can use in the `.vscode/launch.json` file in your Visual Studio Code project folder. 

### Example script
```
#Container specific settings
$ContainerName = "<Container Name>"
$LicenseFile = "<BC License Path + File Name>.bclicense"
$UserName = "<User Name>"
$Password = "<Password>"
$ImportTestToolkit = $true
$IncludeTestLibrariesOnly = $true
$IncludeAL = $true
$MemoryLimit = "16G"

#General settings
$artifactStorageAccount = "bcartifacts" #bcartifacts or bcinsider
$artifactType = "sandbox" #sandbox or onprem
$artifactVersion = "21" #complete or partial version number
$artifactCountryCode = "nl" #country code
$artifactSelect = "latest" #f.i. latest / secondToLastMajor / Closest / All
$sasToken = ""

# Dependent apps
$DependentAppArray = @()

$DependentApp = New-Object -TypeName PSObject
Add-Member -InputObject $DependentApp -MemberType 'NoteProperty' -Name 'AppName' -Value "<Dependent Extension Name 1>"
Add-Member -InputObject $DependentApp -MemberType 'NoteProperty' -Name 'AppFileName' -Value "<Dependent Extension Full File Path 1>.app"
$DependentAppArray += $DependentApp

$DependentApp = New-Object -TypeName PSObject
Add-Member -InputObject $DependentApp -MemberType 'NoteProperty' -Name 'AppName' -Value "<Dependent Extension Name 2>"
Add-Member -InputObject $DependentApp -MemberType 'NoteProperty' -Name 'AppFileName' -Value "<Dependent Extension Full File Path 1>.app"
$DependentAppArray += $DependentApp

#Get artifacts
Write-Host "Artifact storage account: $artifactStorageAccount"
Write-Host "Artifact type: $artifactType"
Write-Host "Artifact select: $artifactSelect"
Write-Host "Artifact country: $artifactcountryCode"
Write-Host "Artifact version: $artifactVersion"

$artifactUrl = Get-BCArtifactUrl -storageAccount $artifactStorageAccount -type $artifactType -version $artifactVersion -country $artifactCountryCode -select $artifactSelect -sasToken $sasToken | Select-Object -First 1
if (-not ($artifactUrl)) {
    throw "Unable to locate artifactUrl from $artifact"
} else {
    Write-Host Using artifactUrl: $artifactUrl
}

#Create the container using NAVContainerHelper
$SecurePassword = ConvertTo-SecureString $Password -AsPlainText -Force
$Credentials = New-Object System.Management.Automation.PSCredential -ArgumentList ($UserName, $SecurePassword)

if ($IncludeAL){
    New-NavContainer -accept_eula -containername $ContainerName -dns 8.8.8.8 -memoryLimit $MemoryLimit -isolation process -artifactUrl $artifactUrl -assignPremiumPlan -credential $Credentials -auth NavUserPassword -accept_outdated -includeAL -updateHosts -doNotCheckHealth
} else {
    New-NavContainer -accept_eula -containername $ContainerName -dns 8.8.8.8 -memoryLimit $MemoryLimit -isolation process -artifactUrl $artifactUrl -assignPremiumPlan -credential $Credentials -auth NavUserPassword -accept_outdated -updateHosts -doNotCheckHealth
}

if ($LicenseFile){
    Import-NavContainerLicense -licenseFile $LicenseFile -containerName $ContainerName
}

Add-FontsToBcContainer -containerName $ContainerName

#Import declared dependent apps
foreach($DependentApp in $DependentAppArray){
    $AppName = $DependentApp.AppName
    $AppFileName = $DependentApp.AppFileName
    Write-Host "";
    Write-Host "Publishing and installing $AppName" -ForegroundColor Yellow
    Publish-NavContainerApp -containerName $ContainerName -appFile $AppFileName -skipVerification
    Sync-NavContainerApp -containerName $ContainerName -appName $AppName
    Install-NavContainerApp -containerName $ContainerName -appName $AppName
}

if ($ImportTestToolkit) {
    if($IncludeTestLibrariesOnly) {
        Import-TestToolkitToNavContainer -containerName $ContainerName -includeTestLibrariesOnly
    } else {
        Import-TestToolkitToNavContainer -containerName $ContainerName 
    }
}
```

[:arrow_left:](../README.md) [Back](../README.md)