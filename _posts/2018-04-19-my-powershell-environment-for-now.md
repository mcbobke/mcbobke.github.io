---
title: My Powershell Environment (For Now)
date: 2018-04-19T11:00:55+00:00
permalink: /2018/04/19/my-powershell-environment-for-now/
categories:
  - PowerShell
tags:
  - automation
  - environment
  - openssh
  - powershell
  - profile
  - windbg
---
My roommate and I have both been diving into scripting in our free time, him mostly in Bash and myself in Powershell. A couple months ago, he mentioned to me that he was proud of his `.bashrc` and walked me through some of its features. I had read about Powershell profile scripts in _Learn Windows Powershell in a Month of Lunches_ and had yet to take a closer look at them. After a fair bit of trial and error to mold my profile into what I want it to be, I'm here to explain its features and the rest of my Powershell environment. You can find the repository [here](https://github.com/mcbobke/Powershell-Environment).

## profile.ps1

With any shell, knowing that you are running with administrative rights is imperative to avoiding catastrophe. A quick Google search resulted in an [easy way](https://social.technet.microsoft.com/Forums/lync/en-US/1ab8cd29-c205-440d-99e9-59ed66943f57/check-whether-the-powershell-is-running-in-elevated-mode-or-no?forum=ITCG) to test this via a short function, which looks like this at the top of my profile:

```powershell
function Test-Administrator {
  $identity = [System.Security.Principal.WindowsIdentity]::GetCurrent()
  $principal = New-Object System.Security.Principal.WindowsPrincipal($identity)
  $admin = [System.Security.Principal.WindowsBuiltInRole]::Administrator
  $IsAdmin = $principal.IsInRole($admin)
  return $IsAdmin
}
```

Next comes the `prompt` function. Whatever is written to the shell within this function will be what is displayed while Powershell waits for the next command. Using the `Test-Administrator` function, I write (ADMINISTRATOR) before anything else if it returns true; there's no mistaking that the shell is running with admin rights. Next, the common username/computer name/working directory string is written, but with color coding to make distinguishing each piece effortless. Finally, the Powershell default greater-than arrow is printed on the next line to allow as much horizontal space for a command as possible. Here's the result in both non-admin and admin mode, respectfully:

![Non-Admin](https://mcbobke.github.io/images/powershell_2018-04-19_00-49-14.png "Non-Admin")
![Admin](https://mcbobke.github.io/images/powershell_2018-04-19_00-48-51.png "Admin")

And here's the code:

```powershell
function prompt {
  # If running as administrator, set the following options
  if (Test-Administrator) {
    Write-Host "(ADMINISTRATOR) " -NoNewline -ForegroundColor Red
  }

  Write-Host "$Env:USERNAME" -NoNewline -ForegroundColor Green
  Write-Host "@" -NoNewline -ForegroundColor DarkGray
  Write-Host "$Env:COMPUTERNAME" -NoNewline -ForegroundColor Magenta
  Write-Host " : " -NoNewline -ForegroundColor DarkGray
  Write-Host "$(Get-Location)".Replace($Env:USERPROFILE, "~") -ForegroundColor Yellow
  return ">" # The prompt function must return a string, or it will write the default prompt
}
```

Finally, I dot-source (effectively the same as importing a function in this shell context) my custom functions, set the window title based on the level of rights the shell has, and extend the environment PATH variable.

```powershell
$psenvPath = "$Env:SystemDrive\psenv"

# Variable to make editing options simpler
$Shell = $Host.UI.RawUI

# Source all of the scripts
foreach ($script in (Get-ChildItem "$psenvPath\scripts\import")) {
  . (Join-Path -Path "$psenvPath\scripts\import" -ChildPath $script)
}

if (Test-Administrator) {
  $Shell.WindowTitle = "Powershell (ADMINISTRATOR)"
}
else {
  $Shell.WindowTitle = "Powershell"
}

# Additonal PATH extension
$env:Path += ";C:\Program Files\OpenSSH"
$env:Path += ";C:\Program Files (x86)\Windows Kits\10\Debuggers\x64"
```

## Installing OpenSSH and WinDbg

Microsoft's work on porting OpenSSH to Windows inspired me to bring it into my Powershell environment. I must admit, I still find myself using Putty regularly due to its Saved Sessions and easy private key authentication setup; my work with OpenSSH has been more experimental than anything, at least until I force myself to switch. [These instructions](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH) are replicated in `Install-WinOpenSSH.ps1`, and reversed in `Uninstall-WinOpenSSH.ps1`.

WinDbg is a tool included in the Windows 10 SDK that can be installed on its own using the following switches:

`win10sdk.exe /features OptionId.WindowsDesktopDebuggers /norestart /ceip off /quiet`

Downloading the setup executable through Powershell proved to be a challenge of parsing the HTML of the download page. `Invoke-WebRequest` conveniently provides a way for all links on the page to be searched through until the desired link display text is found:

```powershell
$downloadUrl = $response `
| Select-Object -ExpandProperty "Links" `
| Where-Object {$PSItem.innerText -eq 'Download the .EXE'} `
| Select-Object -ExpandProperty "href"
```

That download link is then used to retrieve the setup executable, which is stored and used to install WinDbg. If `Uninstall-WinDbg.ps1` is executed, that same executable is used.

## Putting this All Together

`Invoke-EnvironmentSetup.ps1` and `Invoke-EnvironmentTeardown.ps1` are provided for the automated install and uninstall of my environment. I don't think much of anything in these scripts warrants much explanation here as they mostly just copy files around and run the above helper scripts. If there are any specific questions about these scripts or details in the other scripts that I didn't cover, please comment!

## Discoveries and Future Considerations

A couple of key things that I learned:

* Running installers in a script can be problematic if adequate time is not provided for their actions to finish. For example, attempting to delete the executable while it's running will throw an error. A 30-second delay is used when WinDbg is installing or uninstalling to avoid this and other issues.
* Including an check for an existing environment installation in `Invoke-EnvironmentSetup.ps1` made it much easier to rapidly test code changes in the the environment scripts. One less command to run is much more satisfying than it sounds.
* Windows Powershell (Version 5.1) returns a different response than Powershell Core (Version 6.0) does. There's a more technical explanation [here](https://get-powershellblog.blogspot.com/2017/11/powershell-core-web-cmdlets-in-depth.html#L07), but this means that future versions of this script will require revision of the HTML parsing in `Install-WinDbg.ps1`. See my changes to this script in [this](https://github.com/mcbobke/Powershell-Environment/commit/b363f143396f182df8b4a7dc0a8cfb21aa5aa197) 6.0-conversion branch commit.

Questions and ideas that I have for the future:

* Is there a better way to handle the variables in `Invoke-EnvironmentSetup.ps1` and `Invoke-EnvironmentTeardown.ps1` that are marked `$Global:`? I did this so the install/uninstall helper scripts would need script-level variable declarations. It's as clean as I've been able to come up with.
* Is the environment organized well? Am I breaking any unwritten Powershell rules/recommendations by the community?
* My custom functions are currently designed for remote computers only; I need to convert these to work for the local machine if an argument is given to `-ComputerName`.
* Appveyor CI/CD and Pester would be fun to introduce.