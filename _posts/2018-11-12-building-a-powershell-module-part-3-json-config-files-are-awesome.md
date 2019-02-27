---
title: 'Building a PowerShell Module &#8211; Part 3 &#8211; JSON Config Files are Awesome'
date: 2018-11-12T11:28:18+00:00
permalink: /2018/11/12/building-a-powershell-module-part-3-json-config-files-are-awesome/
categories:
  - PowerShell
tags:
  - iperf
  - json
  - powershell
  - psspeedtest
---
JSON support in PowerShell is a beautiful thing. Creating a configuration file for PSSpeedTest to store default iPerf3 server information in is a breeze with the usage of two fairly straightforward functions: `Get-SpeedTestConfig` and `Set-SpeedTestConfig`.

# Part 3 &#8211; JSON Configuration File Getting/Setting

## Get-SpeedTestConfig

The source for `Get-SpeedTestConfig` can be found [here on GitHub](https://github.com/mcbobke/PSSpeedTest/blob/master/PSSpeedTest/public/Get-SpeedTestConfig.ps1), but let&#8217;s dive in to the meat of the function.

[powershell]  
try {  
Write-Verbose -Message "Getting content of config.json and returning as a PSCustomObject."  
$config = Get-Content -Path "$($PSScriptRoot | Split-Path -Parent)\config.json" -ErrorAction "Stop" | ConvertFrom-Json

if ($PassThru){  
return $config  
}  
else {  
Write-Host "Internet Server: $($config.defaultInternetServer.defaultServer)"  
Write-Host "Internet Port: $($config.defaultInternetServer.defaultPort)"  
Write-Host "Local Server: $($config.defaultLocalServer.defaultServer)"  
Write-Host "Local Port: $($config.defaultLocalServer.defaultPort)"  
}  
}  
catch {  
throw "Can&#8217;t find the JSON configuration file. Use &#8216;Set-SpeedTestConfig&#8217; to create one."  
}[/powershell]

First and foremost, there&#8217;s a chance that a configuration file for PSSpeedTest does not already exist; blindly pulling from a path to a JSON file would be bad practice without error handling, so a `try/catch` block is the first thing that should be put in place. Depending on your own thoughts and opinions on error handling, you may want to use `Test-Path` for this purpose rather than `try/catch`.

If a configuration file is present at the expected path and no error is thrown, the rest of the function is simply just reading a text file and converting it to a navigable object with `Get-Content` and `ConvertFrom-Json`.

In the full source of the function you will see that there is a `$PassThru` switch parameter present; if this is specified by the user, the object resulting from `ConvertFrom-Json` will be returned and output will be silenced. This is so that administrators can manipulate this object without directly affecting the stored configuration if they so choose.

## Set-SpeedTestConfig

Source for `Set-SpeedTestConfig` is found [here](https://github.com/mcbobke/PSSpeedTest/blob/master/PSSpeedTest/public/Set-SpeedTestConfig.ps1). Let&#8217;s view this function as three distinct sections; two for error handling and one for finalizing and setting the configuration.

### No Worries About a Non-Existent Configuration

As we discussed before with `Get-SpeedTestConfig`, it&#8217;s never safe to assume that a configuration file already exists. This function is all about setting a config, however, so I&#8217;d rather not throw an error if one doesn&#8217;t exist; I&#8217;d rather create a blank one that will then be set by the calling user.

[powershell]  
try {  
Write-Verbose -Message "Trying Get-SpeedTestConfig before Set-SpeedTestConfig."  
$config = Get-SpeedTestConfig -PassThru  
Write-Verbose -Message "Stored config.json found."  
}  
catch {  
Write-Verbose -Message "No configuration found &#8211; starting with empty configuration."  
$jsonString = @"  
{ "defaultPort" : "5201",  
"defaultLocalServer": {  
"defaultServer" : "",  
"defaultPort" : ""  
},  
"defaultInternetServer" : {  
"defaultServer" : "",  
"defaultPort" : ""  
}  
}  
"@  
$config = $jsonString | ConvertFrom-Json  
}[/powershell]

If the call to `Get-SpeedTestConfig` doesn&#8217;t find an existing configuration file, an empty config is generated via a multi-line JSON string that is piped to `ConvertFrom-Json`. This allows for compatibility with the rest of the function as if a file was found and read with `Get-Content`.

### No Ports Without Servers

Next, it&#8217;s important to consider what combinations of parameters make sense when setting a configuration as well as what configuration elements are already set. Having ports defined without an associated server doesn&#8217;t make sense because there is no default server for iPerf3, of course. Having servers without ports is fine because iPerf3 can run on a default port which is accounted for in my module.

[powershell]  
\# Detailed parameter validation against current configuration  
if ($InternetPort \`  
-and (!($InternetServer)) \`  
-and (!($config.defaultInternetServer.defaultServer))) {  
throw "Cannot set an Internet port with an empty InternetServer setting."  
}  
if ($LocalPort \`  
-and (!($LocalServer)) \`  
-and (!($config.defaultLocalServer.defaultServer))) {  
throw "Cannot set a Local port with an empty LocalServer setting."  
}[/powershell]

If either type of port is being set and an associated server is not also being set or is not pre-existing, an error is thrown.

### Set and Write Out the Object

Finally, the JSON object is modified to store any arguments given by the calling user, converted to a JSON string, and saved in the configuration file.

[powershell]  
if ($InternetServer) {$config.defaultInternetServer.defaultServer = $InternetServer}  
if ($InternetPort) {$config.defaultInternetServer.defaultPort = $InternetPort}  
if ($LocalServer) {$config.defaultLocalServer.defaultServer = $LocalServer}  
if ($LocalPort) {$config.defaultLocalServer.defaultPort = $LocalPort}

Write-Verbose -Message "Setting config.json."  
$config \`  
| ConvertTo-Json \`  
| Set-Content -Path "$($PSScriptRoot | Split-Path -Parent)\config.json"[/powershell]