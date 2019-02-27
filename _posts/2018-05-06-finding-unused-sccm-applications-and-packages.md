---
title: Finding Unused SCCM Applications and Packages
date: 2018-05-06T21:33:49+00:00
permalink: /2018/05/06/finding-unused-sccm-applications-and-packages/
categories:
  - PowerShell
tags:
  - configmgr
  - configurationmanager
  - powershell
  - sccm
  - systemcenter
  - windowspowershell
---
In preparation for an SCCM cleanup project, I was tasked with compiling a list of all Applications and Packages that were not being deployed and had no dependent task sequences or deployment types. Here's how I did it with PowerShell.

# Before we begin: The ConfigMgr Module

Microsoft's `ConfigurationManager` module needs to be installed prior to running this script, or it will fail. You can grab the installer [here](https://www.microsoft.com/en-us/download/details.aspx?id=46681).

# Filter, filter, filter

We need to start with a list of all Applications and Packages and filter down from there. The following code accomplishes this and additionally filters the Applications to our desired results since the objects returned by `Get-CMApplication` provide all the properties that we need to query.

```powershell
# Grab all non-deployed/0 dependent TS/0 dependent DT applications, and all packages (can't filter packages like applications)
$FinalApplications = Get-CMApplication `
| Where-Object {($_.IsDeployed -eq $False) -and ($_.NumberofDependentTS -eq 0) -and ($_.NumberofDependentDTs -eq 0)}
$AllPackages = Get-CMPackage
```

This leaves us with Packages. To achieve the same result as Applications, we will first need a list of all of the existing Task Sequences' `References` to their included items.

```powershell
$TSReferences = Get-CMTaskSequence | Select-Object -ExpandProperty References
```

Next, we need a list of all existing deployments' `PackageID`s.

```powershell
$DeploymentPackageIDs = Get-CMDeployment | Select-Object -ExpandProperty PackageID
```

Finally, we filter `$AllPackages` to only Packages that are not in either of `$TSReferences` and `$DeploymentPackageIDs`.

```powershell
$FinalPackages = New-Object -TypeName 'System.Collections.ArrayList'

# Filter packages to only those that do not have their PackageID in the list of references
foreach ($package in $AllPackages) {
  if (($package.PackageID -notin $TSReferences) -and ($package.PackageID -notin $DeploymentPackageIDs)) {
    $FinalPackages.Add($package)
  }
}
```

And that's it! You can find the full script [here](https://github.com/mcbobke/SCCM-Powershell-Scripts/blob/master/Get-CMApplicationsAndPackagesNoTaskSequences.ps1).