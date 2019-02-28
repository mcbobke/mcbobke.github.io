---
title: 'Building a PowerShell Module - Part 1 - Setting up Plaster'
date: 2018-06-19T09:51:24+00:00
permalink: /2018/06/19/building-a-powershell-module-part-1-setting-up-plaster/
categories:
  - PowerShell
tags:
  - automation
  - invokebuild
  - module
  - pester
  - plaster
  - powershell
  - psdepend
  - psdeploy
---
In this series of blog posts, I'll be detailing how I went about creating [PSSpeedTest](https://github.com/mcbobke/PSSpeedTest), my first publically-released PowerShell module. I want to give **immense** credit to [Kevin Marquette](https://kevinmarquette.github.io/) and [David Christian](https://overpoweredshell.com/) for their blogs about various topics as they really helped me hit the ground running. For this topic specifically, the following blog posts were instrumental and I give full credit to them for utilizing a lot of their ideas:

* [Kevin Marquette - Powershell: Let's build the CI/CD pipeline for a new module](https://kevinmarquette.github.io/2017-01-21-powershell-module-continious-delivery-pipeline/?utm_source=blog&utm_medium=blog&utm_content=titlelink)
* [David Christian - Working With Plaster](https://overpoweredshell.com/Working-with-Plaster/)

Alright, let's talk about Plaster!

# Part 1 - Setting up Plaster

[Plaster](https://github.com/PowerShell/Plaster) is a PowerShell module that provides a means of generating the framework of other PowerShell projects. It utilizes an XML manifest file to generate folder structures, text files, scripts, and really anything that you'd expect to see in a PowerShell project. It can utilize variables defined within template files and the manifest file to fill in values that you specify during project generation, which is very useful during creation of a new PowerShell module for situations such as the need to insert the name of the new module in a bunch of different files. Describing the full suite of features that Plaster provides would be a blog post in and of itself, but thankfully the [about\_Plaster\_CreatingAManifest](https://github.com/PowerShell/Plaster/blob/master/docs/en-US/about_Plaster_CreatingAManifest.help.md) page tells you everything you need to know in great detail. I'll focus on my own personal choices in this post. My Plaster template can be found [here](https://github.com/mcbobke/PlasterModuleTemplate).

## New-PlasterManifest

First and foremost, a valid `plasterManifest.xml` should be generated to ensure that the most basic manifest elements are correctly set. Plaster provides a cmdlet called `New-PlasterManifest` for this exact purpose which allows you to set each possible manifest metadata element. I executed this cmdlet as follows:

```powershell
$manifestParams = @{  
TemplateName = "PlasterModuleTemplate";  
TemplateType = Project;  
Title = "New PowerShell Module Plaster Template";  
Description = "A Plaster template for a new PowerShell module.";  
Author = "Matt Bobke";  
}

New-PlasterManifest @manifestParams
```

This generated the following elements of my manifest:

```text
<metadata>  
<name>PlasterModuleTemplate</name>  
<id>304866b2-7b6f-4ebc-8e99-903751f4722a</id>  
<version>1.0.0</version>  
<title>New PowerShell Module Plaster Template</title>  
<description>A Plaster template for a new PowerShell module.</description>  
<author>Matt Bobke</author>  
<tags></tags>  
</metadata>
```

## Parameters

With the Plaster metadata set, next comes the Parameters that I want to prompt for every time I generate a new module project. I decided that I wanted to prompt for four pieces of information:

* Module Name - The name of the new module.
* Module Description - A short description of the new module.
* Module Version - The version of the new module.
* Module Author - The author of the new module.

The last two parameters seem unnecessary to prompt for, however I wanted to account for the scenarios of a module that currently exists but is being reworked to follow the format of this template (hence prompting for the current version), and a module that is credited to a company or another person (hence prompting for the author). Additionally, both of those parameters have defined default values since they will often be set to `0.0.0` and `Matt Bobke`, respectively.

The Parameters element of the manifest follows the metadata element immediately as follows:

```text
<parameters>  
<parameter name='ModuleName' type='text' prompt='Name of the module' default='${PLASTER_DestinationName}'/>  
<parameter name='ModuleDesc' type='text' prompt='Short description of this module'/>  
<parameter name='ModuleVersion' type='text' default='0.0.0' prompt='Enter the version number for the module'/>  
<parameter name='ModuleAuthor' type='user-fullname' prompt="Module authors' name"/>  
</parameters>
```

## Content

Finally, the meat of the manifest is contained in the Content element. To keep this information organized, I use Plaster's `message` element to divide the Content by type and print messages to the screen during the execution of Plaster that help me identify which type of Content is currently being handled. Whatever is written between `<message>` and `</message>` will be printed to the console.

### Directory Structure

The first step in my Content element is creating the directory structure of the module project. It looks like this:

root  
--modulefolder  
----public  
----private  
--tests

This step also utilizes the `PLASTER_Param_ModuleName` parameter that I prompt for earlier in the manifest. The following lines in the manifest create the directory structure:

```text
<file source='' destination='${PLASTER_PARAM_ModuleName}\public'/>  
<file source='' destination='${PLASTER_PARAM_ModuleName}\private'/>  
<file source='' destination='tests\'/>
```

### Files

Next, all files that are not template files are placed in their proper locations in the generated directory structure:

* `build.Depend.psd1` - The `PSDepend` manifest file that is used to install any other PowerShell modules that a module's build scripts require. (See [PSDepend](https://github.com/RamblingCookieMonster/PSDepend))
* `deploy.PSDeploy.ps1` - The `PSDeploy` script that is used to deploy a module to the PSGallery, and can be altered to support deployment to other repositories or locations. (See [PSDeploy](https://github.com/RamblingCookieMonster/PSDeploy))
* `FunctionTemplate.ps1` - A sample function that purely exists to remind myself of my ideal function structure as I begin work on a module. This gets placed in both the public and private function folders.
* `template.psm1` - A PowerShell script module that is only used to import the work-in-progress module without needing to call the build script. I rarely found myself using this and am not sure that it is a permanent feature of my Plaster template, but it is here for now.
* `module.Build.ps1` - The `Invoke-Build` build script that controls the build pipeline for a module. (See [Invoke-Build](https://github.com/nightroman/Invoke-Build))
* `basicTest.ps1, Help.Tests.ps1, Unit.Tests.ps1` - The beginnings of the `Pester` testing scripts for a module. (See [Pester](https://github.com/pester/Pester))

Explaining each of these files and their associated tools beyond what I've said above would be too much for this blog post alone, and I may write about some of them as parts of this series or other future posts, but there are _plenty_ of great blog posts out there for them already. Here are some examples:

* [Warren Frame - PSDepend: PowerShell Dependencies](http://ramblingcookiemonster.github.io/PSDepend/)
* [Warren Frame - PSDeploy: Simplified PowerShell Based Deployments](http://ramblingcookiemonster.github.io/PSDeploy/)
* [Josh Duffney - Getting Started with Invoke-Build](http://duffney.io/GettingStartedWithInvokeBuild)
* [Josh Duffney - Getting Started with Pester](http://duffney.io/GettingStartedWithPester)

In addition to placing the above files, a PowerShell module manifest is generated using the `newModuleManifest` element, which utilizes all of the Parameters that I prompted for previously. The following lines accomplish all of this:

```text
<newModuleManifest destination='${PLASTER_PARAM_ModuleName}\${PLASTER_PARAM_ModuleName}.psd1' moduleVersion='$PLASTER_PARAM_ModuleVersion' rootModule='${PLASTER_PARAM_ModuleName}.psm1' author='$PLASTER_PARAM_ModuleAuthor' description='$PLASTER_PARAM_ModuleDesc'/>  
<file source='build.Depend.psd1' destination=''/>  
<file source='deploy.PSDeploy.ps1' destination=''/>  
<file source='functions\FunctionTemplate.ps1' destination='${PLASTER_PARAM_ModuleName}\public\FunctionTemplate.ps1'/>  
<file source='functions\FunctionTemplate.ps1' destination='${PLASTER_PARAM_ModuleName}\private\FunctionTemplate.ps1'/>  
<file source='template.psm1' destination='${PLASTER_PARAM_ModuleName}\${PLASTER_PARAM_ModuleName}.psm1'/>  
<file source='module.Build.ps1' destination = '${PLASTER_PARAM_ModuleName}.Build.ps1'/>  
<file source='tests\basicTest.ps1' destination='tests\${PLASTER_PARAM_ModuleName}.Tests.ps1'/>  
<file source='tests\Help.Tests.ps1' destination='tests\Help.Tests.ps1'/>  
<file source='tests\Unit.Tests.ps1' destination='tests\Unit.Tests.ps1'/>
```

### Template Files

Lastly, template files that contain references to Plaster parameters are used to generate files in the module project that contain the expected values in place of those parameters. During execution, plaster recognizes the `templateFile` element as distinct from the `file` element explained above, and expects to find parameters or other supported tokens that it will need to process.

For example, I include the MIT license by default in a new module project and the template license file's copyright line ends with `<%= $PLASTER_PARAM_ModuleAuthor %>`. This reference to the `PLASTER_PARAM_ModuleAuthor` parameter will be replaced with the value stored in it, and the newly-generated file will be moved to its destination.

The following are my current template files and their purposes:

* `LICENSE` - explained above.
* `README.md` - the module project's readme file that is displayed on GitHub or other version control tools.
* `build.ps1` - a wrapper script for the `Invoke-Build` script.
* `appveyor.yml` - the configuration file for the AppVeyor CI/CD tool (to be explained in a future part of this series!).

The following lines fulfill the generation of these template files:

```text
<templateFile source='LICENSE' destination=''/>  
<templateFile source='README.md' destination=''/>  
<templateFile source='build.ps1' destination=''/>  
<templateFile source='appveyor.yml' destination =''/>
```

# Wrapping Up

And there we have it! My Plaster template for a new PowerShell module. You can view the template in full on my [GitHub](https://github.com/mcbobke/PlasterModuleTemplate) repository, and as always, Pull Requests and Issues are encouraged and requested if you so desire. I'm looking forward to the next part of this series where I will detail how I decided on the internal functionality of `PSSpeedTest`. Thanks for reading!