---
title: 'JAMFIT&#8217;s LDAP Sync Script &#8211; Converted from Python 2 to PowerShell'
date: 2018-07-17T10:58:41+00:00
permalink: /2018/07/17/jamfits-ldap-sync-script-converted-from-python-2-to-powershell/
categories:
  - PowerShell
tags:
  - github
  - jamf
  - powershell
  - pull request
---
During the time that I spent on a temporary assignment on Blizzard&#8217;s End User Computing team, work began on implementing [Jamf](https://www.jamf.com/). Being a global company, Blizzard has more than a few office locations and departments that are all represented across Active Directory. A script that Jamf&#8217;s IT team hosts on [GitHub](https://github.com/jamfit/JSS-LDAP-Sync) enables easy syncing from AD to Jamf for these attributes, but it was undesirable to need Python 2 installed in order to run it.

I took on the task of converting the script to PowerShell and it&#8217;s been very useful in our environment so far. It depends on the `ActiveDirectory` module being available, but it&#8217;s fairly safe to assume that Windows administrators have the [RSAT](https://www.microsoft.com/en-us/download/details.aspx?id=45520) tools installed.

You can view the script [here](https://github.com/jamfit/JSS-LDAP-Sync/blob/master/jssldapsync_ps.ps1). The [Pull Request](https://github.com/jamfit/JSS-LDAP-Sync/pull/5) to have this merged into Jamf&#8217;s repository alongside the Python 2 script has been accepted by Jamf, which was super exciting for me as a newbie in the open source community! Let&#8217;s go over the script and my decisions in its design.

## Logging

When I was beginning work on the conversion, there was discussion of scheduling this script to run automatically on some interval to keep AD and the JSS (Jamf Software Server) synced regularly. An addition I made to the features of the Python 2 script is a `Log` function that writes to the console and additionally to a text file if the `LogFile` switch parameter is specified at runtime.

\[powershell\]\[CmdletBinding()\]  
Param(  
[Switch]$LogFile  
)

\# Logging-related variables in script scope  
$Script:LogFile = $LogFile  
$Script:LogFolderPath = ".\Logs"  
$Script:LogFilePath = $LogFolderPath + "\" + (Get-Date -Format FileDate) + ".txt"

function Log ([String]$Content) {  
Write-Host $Content  
if ($Script:LogFile) {Add-Content -Path $Script:LogFilePath -Value $Content}  
}[/powershell]

Only 5 log files will be kept in the designated Logs directory; the script removes the fifth-oldest log before it begins work.

[powershell]# Delete 5th oldest log file (if it exists) and create a new one.  
if ($Script:LogFile) {  
if (!(Test-Path -Path $Script:LogFolderPath)) {  
New-Item -ItemType Directory -Force -Path $Script:LogFolderPath | Out-Null  
}

Get-ChildItem -Path $Script:LogFolderPath | \`  
Where-Object -FilterScript {-not $_.PSIsContainer} | \`  
Sort-Object -Property CreationTime -Descending | \`  
Select-Object -Skip 4 | \`  
Remove-Item -Force | \`  
Out-Null

New-Item -ItemType File -Path $Script:LogFilePath -Force | Out-Null  
}[/powershell]

## Functions

The JSS API operates using REST operations and XML data, so I encapsulated each necessary operation and its accompanied API call in a function to keep the script legible. For the `GetDepartments`/`GetBuildings`, `CreateDepartment`/`CreateBuilding`, and `DeleteDepartment`/`DeleteBuilding` functions, the operations inside each pair of functions are identical except for `department` being replaced with `building` appropriately. I&#8217;ll only be presenting the department-related functions in these cases as to not be repetitive.

### Getting Departments/Buildings from the JSS

The list of departments and buildings in the JSS is not of a fixed length, of course, so I begin each function by instantiating a hash table that can then be appended to with the `Add` method. **Note:** I _really_ should have used `System.Collections.ArrayList` containers for this purpose as they are the most memory-efficient and time-efficient in this situation. Arrays can also be &#8220;appended&#8221; to using the `+=` operator, but this instantiates a whole new array and copies the old one into it which is both memory and time inefficient. Using hash tables necessitated storing each department and building as a key/value pair, with the key being the name of the item and the value being `$null` &#8211; unnecessary and inefficient, but not to the point that any slowness is noticeable during execution.

Next, `Invoke-WebRequest` retrieves the departments/buildings stored in the JSS using the JSS URL and JSS admin credential that the user of the script defines. An attempt is made to cast the content of the response to an `XML` object from which the names of the departments/buildings are extracted; if this cast fails, there were no departments/buildings in the JSS and an empty list is outputted to the PowerShell pipeline. If the cast is successful, the XML hierarchy is iterated through and each department/building name is added to the hash table which is then outputted to the PowerShell pipeline.

[powershell]# Returns a list of all departments in JAMF.  
function GetDepartments ([String]$JssUrl, [PSCredential]$Credential) {  
$listDepartments = @{}  
$requestResponse = Invoke-WebRequest -Uri "$JssUrl/departments" -Credential $Credential

try {  
$root = [Xml]$requestResponse.Content # cast content to XML  
$departmentNodes = $root.SelectNodes("//department")  
}  
catch {  
\# There were no departments stored in JAMF  
$departmentNodes = $null  
}

if ($departmentNodes -ne $null) {  
foreach ($node in $departmentNodes) {  
$listDepartments.Add($node.name, $null)  
}  
}

$listDepartments  
}[/powershell]

### Creating Departments/Buildings in the JSS

Due to the international presence of Blizzard Entertainment, initial testing of the script immediately presented an issue with creating departments and buildings in the JSS whose names contained characters that are invalid in XML. These departments and buildings would be created successfully, but on the next execution of the script the invalid characters would not exist in the JSS listing and consequently would not successfully match any of those in AD, resulting in the immediate deletion of the item from the JSS. I discovered the `[System.Security.SecurityElement]::Escape` static .NET method [here](https://msdn.microsoft.com/en-us/library/system.security.securityelement.escape(v=vs.110).aspx) which replaces any characters that are invalid in XML with their matching escape character. This process, when paired with a `charset=utf-8` designation in the web request `ContentType`, allowed all of the departments and buildings to sync successfully and avoid accidental deletion.

[powershell]# Creates a department in JAMF from the $Name parameter.  
function CreateDepartment ([String]$JssUrl, [PSCredential]$Credential, [String]$Name) {  
$name = [System.Security.SecurityElement]::Escape($Name) # Replaces invalid XML characters  
$body = "<department><name>$name</name></department>" # XML-formatted request body  
$webRequestParams = @{  
Uri = "$JssUrl/departments/id/0";  
Credential = $Credential;  
Method = "Post";  
Body = $body;  
ContentType = &#8216;application/xml; charset=utf-8&#8217;;  
}

$response = Invoke-WebRequest @webRequestParams  
}[/powershell]

### Deleting Departments/Buildings in the JSS

Deletion of departments and buildings did not require any special processing to perform successfully, so the functions are much simpler than those previously described.

[powershell]# Deletes a department from JAMF that matches the $Name parameter.  
function DeleteDepartment ([String]$JssUrl, [PSCredential]$Credential, [String]$Name) {  
$webRequestParams = @{  
Uri = "$JssUrl/departments/name/$Name";  
Credential = $Credential;  
Method = "Delete";  
}

$response = Invoke-WebRequest @webRequestParams  
}[/powershell]

### Querying AD

The list of all unique departments and buildings (street addresses) assigned to user objects in AD is gathered in a fairly straightforward way:

  1. Get all enabled AD objects with the ObjectClass property of &#8220;User&#8221; with the &#8220;Department&#8221; and &#8220;StreetAddress&#8221; properties selected
  2. If a user object has a Department assigned _and_ it is unique, add it to the list of departments
  3. If a user object has a Building assigned _and_ it is unique, add it to the list of buildings

Again, I should have used `System.Collections.ArrayList` containers. Additionally, I notice that I took a concept from the Python 2 script that targets a specific AD OU to search beneath for users, but while the function requires the `$SearchBase` parameter, it doesn&#8217;t actually use it. Our goal was to have all international departments and buildings synced in the JSS, so while this script achieves that, this may not be the case for other companies. This doesn&#8217;t hinder the script, but it prevents users from targeting their user search in a specific OU if they so desire. I&#8217;ll need to implement this or remove the option altogether in a future pull request.

[powershell]# Gathers lists of all unique departments and buildings that exist in LDAP user records.  
function GetLdapLists ([String]$SearchBase, [String]$LdapServer, [PSCredential]$Credential) {  
\# Return variables  
$departments = @{}  
$buildings = @{}

$getADUserParams = @{  
Filter = {(Enabled -eq $True) -and (ObjectClass -eq "User")};  
Properties = "Department", "StreetAddress";  
Server = $LdapServer;  
Credential = $Credential;  
} # $SearchBase is not actually used

$staff = Get-ADUser @getADUserParams

foreach ($user in $staff) {  
try {  
$department = $user.Department  
if (!$departments.ContainsKey($department)) {  
$departments.Add($department, $null)  
}  
}  
catch {  
continue  
}

try {  
$building = $user.StreetAddress  
if (!$buildings.ContainsKey($building)) {  
$buildings.Add($building, $null)  
}  
}  
catch {  
continue  
}  
}

$departments, $buildings  
}[/powershell]

### Compare AD and JSS

In the last bit of data processing, the lists of departments and buildings in AD/JSS are compared against each other which determines which departments/buildings need to be created and deleted. This function is generalized to be used for both departments and buildings. The last case in which I should have used `System.Collections.ArrayList` containers.

[powershell]# Returns:  
\# A list of items that exist in LDAP but not JSS (to be created in JSS).  
\# A list of items that exist in JSS but not LDAP (to be deleted from JSS).  
function CompareLists ([HashTable]$LdapList, [HashTable]$JssList) {  
$toCreate = @{}  
$toDelete = @{}

foreach ($i in $LdapList.Keys) {  
if (!$JssList.ContainsKey($i)) {  
$toCreate.Add($i, $null)  
}  
}

foreach ($i in $JssList.Keys) {  
if (!$LdapList.ContainsKey($i)) {  
$toDelete.Add($i, $null)  
}  
}

$toCreate, $toDelete  
}[/powershell]

## Execution

With all of the functions defined, execution can begin! Since most of the area below line 190 `# Begin script execution` are simply calls to the functions described above or variables that need to be filled in by the user, I&#8217;ll explore two areas of interest that have yet to be explained.

### Optional Stored Credentials

The Python 2 script provided variables that are designed to store the username and password of the JSS and AD accounts that will be used to execute this script and read/write data from both directories. The PowerShell script prompts for these values by default, but I left a block of commented code that can be uncommented and used to store these credentials explicitly if desired.

[powershell]# LDAP/JSS Credentials &#8211; assumed to be the same  
\# Uncomment the below block and enter username/password for autorun  
<# $Username = ""  
$PasswordUnencrypted = ""  
$SecureStringParams = @{  
String = $PasswordUnencrypted;  
AsPlainText = $True;  
Force = $True;  
}  
$Password = ConvertTo-SecureString @SecureStringParams # Enforcing SecureString type for password #>[/powershell]

### Optional CSV Output

As an alternative to logging in a text file, a block of commented code at the end of the script can be uncommented to output four CSV files containing the lists of departments/buildings to be created/deleted.

[powershell]# CSV Output &#8211; uncomment to output lists of departments/buildings created/deleted  
<# $JssCreateDepartments.GetEnumerator() | Export-Csv -Path ".\CreateDepartments.csv" -Delimiter &#8216;,&#8217; -NoTypeInformation  
$JssDeleteDepartments.GetEnumerator() | Export-Csv -Path ".\DeleteDepartments.csv" -Delimiter &#8216;,&#8217; -NoTypeInformation  
$JssCreateBuildings.GetEnumerator() | Export-Csv -Path ".\CreateBuildings.csv" -Delimiter &#8216;,&#8217; -NoTypeInformation  
$JssDeleteBuildings.GetEnumerator() | Export-Csv -Path ".\DeleteBuildings.csv" -Delimiter &#8216;,&#8217; -NoTypeInformation #>[/powershell]

## Summary

Not considering my inefficient use of hash tables and unused `$SearchBase` parameter, this script is a successful modern approach to syncing the departments and buildings in a JSS with AD. The two described oversights will be remedied and a pull request will be submitted at a later date. I&#8217;m thankful to the JAMFIT team for merging my conversion of their Python 2 script!