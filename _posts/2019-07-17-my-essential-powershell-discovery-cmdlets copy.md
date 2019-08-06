---
title: 'My Essential PowerShell Discovery Cmdlets'
date: 2019-07-17
permalink: /2019/07/17/my-essential-powershell-discovery-cmdlets/
categories:
  - PowerShell
  - Basics
tags:
  - powershell
  - basics
---
When I'm doing daily work in the shell, there are a number of cmdlets and associated arguments that I find myself typing often to discover new things and re-discover things that I've previously encountered. Maybe getting these written out with explanations and examples will help some of you while you learn PowerShell, however I do believe that these are useful regardless of your level of experience. You really never stop learning (and definitely never stop forgetting). There will always be situations where you need to go back to the basics, so to speak.

## Get-Help

If there's one cmdlet that you should memorize the name of, it's `Get-Help`. It's your key to understanding any single cmdlet and many of the advanced PowerShell concepts (provided that a useful and up-to-date help file is available). If you've never read help files in the console window before, start by opening an admin shell and running `Update-Help -ErrorAction SilentlyContinue`. I specify the `SilentlyContinue` behavior on errors because if there are cmdlets that do not have available sources for help files, PowerShell will throw an error and continue updating those help files that it does have a source for. I'd rather not clutter the shell with those errors. Even Microsoft lets some cmdlets go without available help files!

Now that your help files are up to date, you can try out `Get-Help` with some of the cmdlets that I'll be explaining further on in this post. It will display the help file for that cmdlet in the console.

```text
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> Get-Help -Name "Get-Member"

NAME
    Get-Member

SYNOPSIS
    Gets the properties and methods of objects.


SYNTAX
    Get-Member [[-Name] <String[]>] [-Force] [-InputObject <PSObject>] [-MemberType {AliasProperty | CodeProperty | Property | NoteProperty |
    ScriptProperty | Properties | PropertySet | Method | CodeMethod | ScriptMethod | Methods | ParameterizedProperty | MemberSet | Event | Dynamic |
    All}] [-Static] [-View {Extended | Adapted | Base | All}] [<CommonParameters>]


DESCRIPTION
    The Get-Member cmdlet gets the members, the properties and methods, of objects.

...
```

### -Full

The `-Full` switch for `Get-Help` is my first choice for cmdlets that I've 1) never used before or 2) haven't used in a considerable amount of time. This returns the entirety of the help file for a single cmdlet which is then scrollable with Page Up/Page Down, Up/Down arrows, and the mouse wheel if you're in a GUI environment. It's very convenient to avoid the need to alt-tab into a web browser, especially on laptops.

![Get-Help -Name "Get-Member" -Full](/images/gethelp_full.gif "Get-Help -Name 'Get-Member' -Full")

### -Examples

The `-Examples` switch for `Get-Help` is useful for skipping directly to any examples of cmdlet usage that are provided by the author. I tend to use this with cmdlets that I have used before but need a quick refresher on syntax. One in particular is `Get-ADComputer` from the ActiveDirectory module, I can never remember the syntax for the `-Filter` parameter off the top of my head.

```text
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> Get-Help -Name 'Get-ADComputer' -Examples

NAME
    Get-ADComputer

SYNOPSIS
    Gets one or more Active Directory computers.
...
    -------------------------- EXAMPLE 2 --------------------------

    C:\PS>Get-ADComputer -Filter 'Name -like "Fabrikam*"' -Properties IPv4Address | FT Name,DNSHostName,IPv4Address -A

    name          dnshostname                ipv4address
    ----          -----------                -----------
    FABRIKAM-SRV1 FABRIKAM-SRV1.Fabrikam.com 10.194.99.181
    FABRIKAM-SRV2 FABRIKAM-SRV2.Fabrikam.com 10.194.100.37

    Description

    -----------

    Get all the computers with a name starting by a particular string and showing the name, dns hostname and IPv4 address.
...
```

### -Online

The `-Online` switch for `Get-Help` displays the online (web-based) version of the help information in your computer's default Internet browser. This is great for quickly firing up a new tab in a running browser session that navigates directly to the help information for the cmdlet in question, which can be a better experience than reading help directly in the PowerShell Console at times. However, this only works if that online help exists in the first place.

Microsoft gives some information about this in their own help information for `Get-Help` itself (found by running `Get-Help Get-Help -Parameter 'Online'`):

```text
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> Get-Help -Name 'Get-Help' -Parameter 'Online'

-Online [<SwitchParameter>]
    Displays the online version of a help topic in the default Internet browser. This parameter is valid only for
    cmdlet, function, workflow and script help topics. You cannot use the Online parameter in Get-Help commands in a
    remote session.

    For information about supporting this feature in help topics that you write, see about_Comment_Based_Help
    (http://go.microsoft.com/fwlink/?LinkID=144309), and Supporting Online Help
    (http://go.microsoft.com/fwlink/?LinkID=242132), and How to Write Cmdlet
    Helphttp://go.microsoft.com/fwlink/?LinkID=123415 (http://go.microsoft.com/fwlink/?LinkID=123415) in the Microsoft
    Developer Network MSDN library.

    Required?                    true
    Position?                    named
    Default value                False
    Accept pipeline input?       False
    Accept wildcard characters?  false
```

Unfortunately, the link to the _Supporting Online Help_ article is dead as of Windows 10 version 1903, but it can be found here: [Supporting Online Help](https://docs.microsoft.com/en-us/powershell/developer/module/supporting-online-help)

## Get-Command

`Get-Command` is my next-most-often-run PowerShell discovery cmdlet. If I'm drawing a blank on the full name of a cmdlet but I know that I remember at least a small portion of the name, I'm usually able to find it with this cmdlet and then do any further discovery necessary - such as `Get-Help`! `Get-Command` gets any and all cmdlets, aliases, functions, workflows, filters, scripts, and applications installed on your computer. These can be filtered using its parameters.

For example, you can see every single cmdlet starting with `Get-` by running the following:

```text
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> Get-Command -Name 'Get-*'

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           Get-ADUserSnapshot                                 0.0.81     PSSharedGoods
Alias           Get-AG2ApiLis                                      3.3.542.0  AWSPowerShell
Alias           Get-AppPackage                                     2.0.1.0    Appx
Alias           Get-AppPackageDefaultVolume                        2.0.1.0    Appx
Alias           Get-AppPackageLastError                            2.0.1.0    Appx
Alias           Get-AppPackageLog                                  2.0.1.0    Appx
Alias           Get-AppPackageManifest                             2.0.1.0    Appx
Alias           Get-AppPackageVolume                               2.0.1.0    Appx
Alias           Get-AppProvisionedPackage                          3.0        Dism
...
```

The wildcard `*` character is supported by many of the parameters as shown in this example.

### -Verb and -Noun

`-Name` is a good place to start with `Get-Command` but sometimes a more finely-tuned search is desired, and that's where `-Verb` and `-Noun` come in. The PowerShell command naming convention specifies a 'Verb-Noun' syntax which is what each parameter will match on respectively.

A recent personal anecdote for this would be work that I've been doing with the AWSPowerShell module. [AWS EC2](https://aws.amazon.com/ec2/) has a laundry list of available cmdlets in the AWSPowerShell module that all have nouns starting with `EC2`. I was working on gathering some data about existing EC2 Instances and wanted to know what cmdlets were available that had `Get` as their verb, that were also related to EC2.

```text
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> Get-Command -Verb 'Get' -Noun 'EC2*'

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           Get-EC2AccountAttributes                           3.3.542.0  AWSPowerShell
Alias           Get-EC2ExportTasks                                 3.3.542.0  AWSPowerShell
Alias           Get-EC2FlowLogs                                    3.3.542.0  AWSPowerShell
Alias           Get-EC2Hosts                                       3.3.542.0  AWSPowerShell
Alias           Get-EC2ReservedInstancesModifications              3.3.542.0  AWSPowerShell
Alias           Get-EC2VpcPeeringConnections                       3.3.542.0  AWSPowerShell
Cmdlet          Get-EC2AccountAttribute                            3.3.542.0  AWSPowerShell
Cmdlet          Get-EC2Address                                     3.3.542.0  AWSPowerShell
Cmdlet          Get-EC2AggregateIdFormat                           3.3.542.0  AWSPowerShell
Cmdlet          Get-EC2AvailabilityZone                            3.3.542.0  AWSPowerShell
Cmdlet          Get-EC2BundleTask                                  3.3.542.0  AWSPowerShell
Cmdlet          Get-EC2ByoipCidr                                   3.3.542.0  AWSPowerShell
...
```

The inclusion of the wildcard filters the list down to only cmdlets whose nouns start with 'EC2'. This allowed me to easily find what I needed.

### -Module

Say you just installed a new module and you'd like to know what cmdlets are provided to you as a user from that single module. Try the `-Module` parameter:

```text
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> Get-Command -Module 'PSSpeedTest'

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        Get-SpeedTestConfig                                2.0.1      PSSpeedTest
Function        Install-SpeedTestServer                            2.0.1      PSSpeedTest
Function        Invoke-SpeedTest                                   2.0.1      PSSpeedTest
Function        Remove-SpeedTestServer                             2.0.1      PSSpeedTest
Function        Set-SpeedTestConfig                                2.0.1      PSSpeedTest
```

Seeing a complete list of all cmdlets available in a module gives you a good idea of what you can accomplish with it.

## Get-Module

`Get-Module` informs you of every module that has been imported in the current PowerShell session. The import may have been done explicitly with `Import-Module` or automatically by using a cmdlet from a module that had not yet been imported.

```text
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> Get-Module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     0.0        chocolateyProfile                   {TabExpansion, Update-SessionEnvironment, refreshenv}
Binary     1.0.0.0    Microsoft.PowerShell.LocalAccounts  {Add-LocalGroupMember, Disable-LocalUser, Enable-LocalUser...
Manifest   3.1.0.0    Microsoft.PowerShell.Management     {Add-Computer, Add-Content, Checkpoint-Computer, Clear-Con...
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Object...}
Script     1.0.0      posh-git                            {Add-PoshGitToProfile, Expand-GitCommand, Format-GitBranch...
Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PS...
Script     2.0.1      PSSpeedTest                         {Get-SpeedTestConfig, Install-SpeedTestServer, Invoke-Spee...
```

I don't find myself using `Get-Module` in this way very often because I don't usually care what has been imported up to this point, but this _can_ be useful for debugging purposes such as ensuring that the expected version of any given module has been loaded.

Adding in the `-ListAvailable` switch parameter tells `Get-Module` to search for all modules that are installed at directory paths listed in the `PSModulePath` environment variable. Try seeing what your `PSModulePath` variable contains:

```text
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> $Env:PSModulePath
C:\Users\Matt\Documents\WindowsPowerShell\Modules;C:\Program Files\WindowsPowerShell\Modules;C:\WINDOWS\system32\Windows
PowerShell\v1.0\Modules;C:\Program Files\Intel\
```

Now see every installed module in these directories:

```text
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> Get-Module -ListAvailable


    Directory: C:\Users\Matt\Documents\WindowsPowerShell\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Binary     3.3.542.0  AWSPowerShell                       {Clear-AWSHistory, Set-AWSHistoryConfiguration, Initialize...
Script     2.0.9      BuildHelpers                        {Add-TestResultToAppveyor, Export-Metadata, Find-NugetPack...
Script     2.0.8      BuildHelpers                        {Add-TestResultToAppveyor, Export-Metadata, Find-NugetPack...
Script     1.2.0      BuildHelpers                        {Add-TestResultToAppveyor, Export-Metadata, Find-NugetPack...
Script     1.1.4      BuildHelpers                        {Add-TestResultToAppveyor, Export-Metadata, Find-NugetPack...
Script     1.0.0.1    ChocolateyGet                       Compare-SemVer
Script     0.0.2      Connectimo                          {Connect-WinAzure, Connect-WinAzureAD, Connect-WinConnecti...
Script     6.2.1      ImportExcel                         {Add-ConditionalFormatting, Add-ExcelChart, Add-ExcelDataV...
Script     6.0.0      ImportExcel                         {Add-ConditionalFormatting, Add-ExcelChart, Add-ExcelDataV...
Script     5.1.1      ImportExcel                         {ColorCompletion, Send-SQLDataToExcel, Join-Worksheet, Exp...
Script     4.0.13     ImportExcel                         {Import-UPS, ConvertFrom-ExcelSheet, Test-Number, Get-Rang...
Script     5.5.2      InvokeBuild                         {Invoke-Build, Build-Checkpoint, Build-Parallel}
...
```

This will not only show every module that is installed that PowerShell knows about, it also shows every version installed in the event that you have multiple versions of a single module.

## Get-InstalledModule

`Get-InstalledModule` is specific to the module [PowerShellGet](https://github.com/PowerShell/PowerShellGet), which is really a package manager for PowerShell itself. It allows you to install community and non-default Microsoft PowerShell modules from sources such as [The PowerShell Gallery](https://www.powershellgallery.com/).

PowerShellGet comes included with:

* Windows 10
* Windows Server 2016 or newer
* Windows Management Framework (WMF) 5.0 or newer
* PowerShell 6 (otherwise known as PowerShell Core)

`Get-InstalledModule` will display every module that was installed on your computer by PowerShellGet.

```text
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> Get-InstalledModule

Version              Name                                Repository           Description
-------              ----                                ----------           -----------
1.4.2                PackageManagement                   PSGallery            PackageManagement (a.k.a. OneGet) is a new way to discover and install software packages from around the web....
1.0.0-beta3          posh-git                            PSGallery            Provides prompt with Git status summary information and tab completion for Git commands, parameters, remotes and branch names.
2.1.5                PowerShellGet                       PSGallery            PowerShell module with commands for discovering, installing, updating and publishing the PowerShell artifacts like Modules, DSC Resources, Role Capabilities and Scripts.
2.0.1                PSSpeedTest                         PSGallery            A module for testing network bandwidth over the internet as well as private networks.
3.3.542.0            AWSPowerShell                       PSGallery            The AWS Tools for Windows PowerShell lets developers and administrators manage their AWS services from the Windows PowerShell scripting environment.
2.0.9                BuildHelpers                        PSGallery            Helper functions for PowerShell CI/CD scenarios.
1.0.0.1              ChocolateyGet                       PSGallery            An PowerShell OneGet provider that discovers packages from https://www.chocolatey.org.
0.0.2                Connectimo                          PSGallery            Simple connectivity project
6.2.1                ImportExcel                         PSGallery            PowerShell module to import/export Excel spreadsheets, without Excel....
5.5.2                InvokeBuild                         PSGallery            Build and test automation in PowerShell
4.8.1                Pester                              PSGallery            Pester provides a framework for running BDD style Tests to execute and validate PowerShell commands inside of PowerShell and offers a powerful set of Mocking Functions that allow tests to m...
1.1.3                Plaster                             PSGallery            Plaster scaffolds PowerShell projects and files.
0.2.3                PSDepend                            PSGallery            PowerShell Dependency Handler
1.0.2                PSDeploy                            PSGallery            Module to simplify PowerShell based deployments
1.16.1               PSScriptAnalyzer                    PSGallery            PSScriptAnalyzer provides script analysis and checks for potential code defects in the scripts by applying a group of built-in or customized rules on the scripts being analyzed.
0.0.81               PSSharedGoods                       PSGallery            Module covering functions that are shared within multiple projects
0.85                 PSWriteColor                        PSGallery            Write-Color is a wrapper around Write-Host allowing you to create nice looking scripts, with colorized output. It provides easy manipulation of colors, logging output to file (log) and nice...
0.0.45               PSWriteHTML                         PSGallery            Module that allows creating HTML content/reports in easy way.
```

Running `Get-InstalledModule` with no parameters will only show you the most recent version of each installed module. If you'd like to see all installed versions, you will need to manipulate the output via the pipeline. The `-AllVersions` parameter does not work with wildcards, so each individual module name must be piped back to `Get-InstalledModule`.

```text
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> Get-InstalledModule | Select-Object -ExpandProperty 'Name' | ForEach-Object -Process {Get-InstalledModule -Name $_ -AllVersions}

Version              Name                                Repository           Description
-------              ----                                ----------           -----------
1.3.2                PackageManagement                   PSGallery            PackageManagement (a.k.a. OneGet) is a new way to discover and install software packages from around the web....
1.4                  PackageManagement                   PSGallery            PackageManagement (a.k.a. OneGet) is a new way to discover and install software packages from around the web....
1.4.2                PackageManagement                   PSGallery            PackageManagement (a.k.a. OneGet) is a new way to discover and install software packages from around the web....
0.7.3                posh-git                            PSGallery            Provides prompt with Git status summary information and tab completion for Git commands, parameters, remotes and branch names.
1.0.0-beta3          posh-git                            PSGallery            Provides prompt with Git status summary information and tab completion for Git commands, parameters, remotes and branch names.
2.1.3                PowerShellGet                       PSGallery            PowerShell module with commands for discovering, installing, updating and publishing the PowerShell artifacts like Modules, DSC Resources, Role Capabilities and Scripts.
2.1.4                PowerShellGet                       PSGallery            PowerShell module with commands for discovering, installing, updating and publishing the PowerShell artifacts like Modules, DSC Resources, Role Capabilities and Scripts.
2.1.5                PowerShellGet                       PSGallery            PowerShell module with commands for discovering, installing, updating and publishing the PowerShell artifacts like Modules, DSC Resources, Role Capabilities and Scripts.
2.0.1                PSSpeedTest                         PSGallery            A module for testing network bandwidth over the internet as well as private networks.
3.3.542.0            AWSPowerShell                       PSGallery            The AWS Tools for Windows PowerShell lets developers and administrators manage their AWS services from the Windows PowerShell scripting environment.
1.1.4                BuildHelpers                        PSGallery            Helper functions for PowerShell CI/CD scenarios.
...
```

## Find-Module

`Find-Module` searches all known PowerShell module repositories for a desired module, most often for a name that you specify via the `-Name` parameter. This parameter supports wildcards.

```text
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> Find-Module -Name '*AWS*'

Version              Name                                Repository           Description
-------              ----                                ----------           -----------
3.3.542.0            AWSPowerShell                       PSGallery            The AWS Tools for Windows PowerShell lets developers and administrators manage their AWS services from the Windows PowerShell scripting environment.
3.3.542.0            AWSPowerShell.NetCore               PSGallery            The AWS Tools for PowerShell Core lets developers and administrators manage their AWS services from the PowerShell Core scripting environment.
1.2.0.0              AWSLambdaPSCore                     PSGallery            The AWS Lambda Tools for Powershell can be used to create and deploy AWS Lambda functions written in PowerShell.
0.9.1.326            ACMESharp.Providers.AWS             PSGallery            AWS Provider extension library for ACMESharp Client.
0.5.0.0              AwsDscToolkit                       PSGallery            Registers an AWS EC2 instance as a DSC Node in Azure Automation
1.2.58               AWSWindowsHelpers                   PSGallery            Helper functions for working with Windows Instances on AWS
...
```

You can then pipe the results to `Install-Module` to install all discovered modules, but be careful with this since it could be a very large amount.

## Get-Member

All of the previous cmdlets that I've discussed have had to do with discovering cmdlets within modules, information about those cmdlets, and modules themselves. What about the objects that are returned by many of the cmdlets that you use? What properties and methods are available with those objects?

Suppose you have a string that you stored in a variable after using `Get-Content` to get the contents of a text file. You may not know that there are many non-obvious properties and methods available from that string that can be used to get information from it as well as manipulate it. Or, you may be trying to remember what the name of that one particular property or method is.

This is where `Get-Member` comes in. Throw that variable on the pipeline and pipe it to `Get-Member`!

```text
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> $text = Get-Content .\text.txt
(ADMINISTRATOR) Matt@MBOBKE-DT : E:\Git\mcbobke.github.io [≡ discoverypost]
> $text | Get-Member


   TypeName: System.String

Name             MemberType            Definition
----             ----------            ----------
Clone            Method                System.Object Clone(), System.Object ICloneable.Clone()
CompareTo        Method                int CompareTo(System.Object value), int CompareTo(string strB), int IComparable.CompareTo(System.Object obj), int IComparable[string].CompareTo(string other)
Contains         Method                bool Contains(string value)
CopyTo           Method                void CopyTo(int sourceIndex, char[] destination, int destinationIndex, int count)
EndsWith         Method                bool EndsWith(string value), bool EndsWith(string value, System.StringComparison comparisonType), bool EndsWith(string value, bool ignoreCase, cultureinfo culture)
Equals           Method                bool Equals(System.Object obj), bool Equals(string value), bool Equals(string value, System.StringComparison comparisonType), bool IEquatable[string].Equals(string other)
GetEnumerator    Method                System.CharEnumerator GetEnumerator(), System.Collections.IEnumerator IEnumerable.GetEnumerator(), System.Collections.Generic.IEnumerator[char] IEnumerable[char].GetEnumerator()
GetHashCode      Method                int GetHashCode()
GetType          Method                type GetType()
...
```

These properties and methods can be accessed by using dot notation:

```powershell
$text.Clone()
$text.GetType()
```

## Summary

PowerShell is huge and it's easy to find yourself in a situation where you're digging around for that one piece of information that will carry you on to the next step. These cmdlets and parameters that I've detailed only truly scratch the surface of what PowerShell provides for you in terms of discovery. In fact, much of the information you may ever need can be had right in the console itself using these cmdlets and others like them, no web browser required. I encourage you to look at the other parameters that are available to these cmdlets and discover more tools that you can use to discover other tools (yes, you read that correctly!).
