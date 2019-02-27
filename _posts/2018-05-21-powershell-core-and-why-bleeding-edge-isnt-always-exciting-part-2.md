---
title: PowerShell Core and Why Bleeding-Edge Isn’t Always Exciting – Part 2
date: 2018-05-21T09:22:33+00:00
author: Matt Bobke
permalink: /2018/05/21/powershell-core-and-why-bleeding-edge-isnt-always-exciting-part-2/
categories:
  - PowerShell
tags:
  - powershell
  - powershellcore
  - ps6
  - pscore
---
Following up on my [last post](https://mattbobke.com/2018/04/30/powershell-core-and-why-bleeding-edge-isnt-always-exciting-part-1/) regarding PowerShell Core, my open pull request with the PowerShell team has been merged and I&#8217;m ready to write about it!

I want to additionally expand on the title of this post as well as the previous. PowerShell is exciting, and any bugs or unexpected behaviors that I encounter don&#8217;t change that; what I mean by &#8220;isn&#8217;t always exciting&#8221; is that sometimes a bit of extra research (or, in this case, some bugfixes) may be required to get to the end goal of the code that I&#8217;m writing.

## &#8230;Where did that item go?

When I first installed PowerShell Core, I attempted to install my [PowerShell Environment](https://github.com/mcbobke/Powershell-Environment) to see if it was immediately compatible between versions 5.1 and 6.0. A few different items in the setup script depend on the directory `C:\psenv` existing, and I saw multiple exceptions thrown stating that this directory didn&#8217;t exist. This was strange because I explicitly create the directory if it doesn&#8217;t already exist. While investigating with Windows Explorer, I realized that the `psenv` directory had been created in the root of my environment repository directory despite specifying the root of `$Env:SystemDrive` explicitly:

[powershell]New-Item -Path "$Env:SystemDrive\" -Name "psenv" -ItemType "Directory" -Force[/powershell]

Upon further investigation with the `New-Item` and `Get-Item` cmdlets, I realized that any new item created with a path that includes a drive letter followed by a colon would end up in the current working directory. This can be reproduced using the steps in [this](https://github.com/PowerShell/PowerShell/issues/5228#event-1630856720) GitHub issue.

[powershell]Set-Location ~  
New-Item -Path C:\ -Name Test -ItemType Directory -Verbose[/powershell]

[powershell]PS C:\Users\mbobke> Get-Item C:\Test  
Get-Item : Cannot find path &#8216;C:\Test&#8217; because it does not exist.  
At line:1 char:1  
+ Get-Item C:\Test  
+ ~~~~~~~~~~~~~~~~  
+ CategoryInfo : ObjectNotFound: (C:\Test:String) [Get-Item], ItemNotFoundException  
+ FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetItemCommand

PS C:\Users\mbobke> Get-Item ~\Test

Directory: C:\Users\mbobke

Mode LastWriteTime Length Name  
&#8212;- &#8212;&#8212;&#8212;&#8212;- &#8212;&#8212; &#8212;-  
d&#8212;&#8211; 25/10/2017 13:51 Test[/powershell]

The same bug would not occur if the given path is a UNC path, nor if the path is one or more directory levels deeper than the root:

[powershell]PS C:\Users\mbobke> New-Item -Path "\\$env:COMPUTERNAME\c$" -Name "psenv" -ItemType "Directory"

Directory: \\MBOBKE-DT\c$

Mode LastWriteTime Length Name  
&#8212;- &#8212;&#8212;&#8212;&#8212;- &#8212;&#8212; &#8212;-  
d&#8212;&#8211; 4/3/2018 5:27 PM psenv

PS C:\Users\mbobke> New-Item -Path "C:\temp" -Name "psenv" -ItemType "Directory"

Directory: C:\temp

Mode LastWriteTime Length Name  
&#8212;- &#8212;&#8212;&#8212;&#8212;- &#8212;&#8212; &#8212;-  
d&#8212;&#8211; 4/3/2018 5:32 PM psenv[/powershell]

## It&#8217;s Bug-Squashin&#8217; Time

At this point, I realized that I wasn&#8217;t crazy and this had to be a bug in PowerShell Core itself. (Trust me, I `New-Item`&#8216;d that directory more times than I&#8217;d like to admit&#8230;) I am admittedly not experienced with C#, at least beyond what you utilize writing beginner Unity3D scripts. However, I was bound and determined to fix this bug myself no matter how many Google search tabs I&#8217;d have to open or debugging sessions I&#8217;d have to start.

I cloned the PowerShell repository and opened it in Visual Studio Code, then searched the `src/` directory for anything that I could find related to `New-Item`. I shortly came across [this method](https://github.com/PowerShell/PowerShell/blob/master/src/System.Management.Automation/namespaces/FileSystemProvider.cs#L2075) which seemed to be either an entry point or helper method for the cmdlet in question; I placed a breakpoint there and started a debugging session. Executing `New-Item` resulted in the execution pausing at that line, and I was then able to step through the code. Visual Studio Code has a nice debug variable explorer that helped me find exactly where the `Path` value was being evaluated.

<img src="https://mattbobke.com/wp-content/uploads/2018/05/Code_2018-05-21_00-09-47.png" alt="" width="463" height="203" class="alignnone size-full wp-image-222" srcset="https://mattbobke.com/wp-content/uploads/2018/05/Code_2018-05-21_00-09-47.png 463w, https://mattbobke.com/wp-content/uploads/2018/05/Code_2018-05-21_00-09-47-300x132.png 300w" sizes="(max-width: 463px) 100vw, 463px" /> 

I paid careful attention to the `path` variable while stepping into each stack of method calls and noticed that it finally began to change in `src/System.Management.Automation/namespaces/LocationGlobber.cs`, so I focused my attention here. The `GetDriveRootRelativePathFromPSPath` method parses the path to determine whether or not it is absolute or relative; in our case, it&#8217;s absolute, so it then strips the drive letter, colon, and the following path separator (slash) from the path. The path is now empty, and the method goes on to incorrectly determine that the path given by the user is empty and therefore uses the current working directly. To fix this, I added [checks](https://github.com/PowerShell/PowerShell/pull/6600/files) to determine if the path became empty after it had been parsed, and if so to use the root of the drive in the path as the destination path. I wrote a couple of tests, submitted a pull request, and it was merged after some refinement with the help of other contributors and the PowerShell team.

This was my first open-source contribution ever, and it is pretty cool to see that I helped make something that I&#8217;m so passionate about better!

Bonus points: I also introduced a bug with these changes where using `Set-Location` to change to a drive on the machine that had been previously visited would fail to restore the previous working directory of that drive. You can find the related issue as well as the pull request where I fixed that issue below.

## Relevant Links

  * Issue filed for items being created at current working directory unexpectedly: [#5228](https://github.com/PowerShell/PowerShell/issues/5228#event-1630856720)
  * Pull Request to fix #5228: [#6600](https://github.com/PowerShell/PowerShell/pull/6600)
  * Issue filed for bug that was created because of my changes in above Pull Request: [#6749](https://github.com/PowerShell/PowerShell/issues/6749#event-1630856719)
  * Pull Request to fix #6749: [#6774](https://github.com/PowerShell/PowerShell/pull/6774)