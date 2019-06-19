---
title: 'My Essential PowerShell Discovery Cmdlets'
date: 2019-05-21
permalink: /2019/05/21/my-essential-powershell-discovery-cmdlets/
categories:
  - PowerShell
  - Basics
tags:
  - powershell
  - basics
---
When I'm doing daily work in the shell, there are a number of cmdlets and associated arguments that I find myself typing often to discover new things and re-discover topics that I've previously covered. Maybe getting these written out with explanations and examples will help some of you while you learn PowerShell, however I do believe that these are useful regardless of your level of experience. You really never stop learning (and definitely never stop forgetting). There will always be situations where you need to go back to the basics, so to speak.

## Get-Help

If there's one cmdlet that you should memorize the name of, it's `Get-Help`. It's your key to understanding any single cmdlet and many of the advanced PowerShell concepts (provided that a useful and up-to-date help file is available). If you've never read help files in the console window before, start by opening an admin shell and running `Update-Help -ErrorAction SilentlyContinue`. I specify the `SilentlyContinue` behavior on errors because if there are cmdlets that do not have available sources for help files, PowerShell will throw an error and continue updating those help files that it does have a source for. I'd rather not clutter the shell with those errors. Even Microsoft lets some cmdlets go without available help files!

Now that your help files are up to date, you can try out `Get-Help` with some of the cmdlets that I'll be explaining further on in this post.

```text
(ADMINISTRATOR) mbobke@IR-MBOBKE-WD : ~\Documents\Git\mcbobke.github.io
>Get-Help -Name "Get-Member"

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

### -Online

## Get-Command

### -Module

## Get-Module

### -ListAvailable

## Get-InstalledModule

## Get-Member