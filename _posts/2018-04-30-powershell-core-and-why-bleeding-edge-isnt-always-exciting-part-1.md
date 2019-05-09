---
title: PowerShell Core and Why Bleeding-Edge Isn't Always Exciting - Part 1
date: 2018-04-30T00:28:33+00:00
permalink: /2018/04/30/powershell-core-and-why-bleeding-edge-isnt-always-exciting-part-1/
categories:
  - PowerShell
tags:
  - powershell
  - powershellcore
  - ps6
  - pscore
---

When I learned about [PowerShell Core](https://blogs.msdn.microsoft.com/powershell/2018/01/10/powershell-core-6-0-generally-available-ga-and-supported/), I was a few months into diving deep into Windows PowerShell. I had already written a few scripts that I would consider "advanced" in some way, one of which being an earlier version of my script to [install WinDbg from the Win10 SDK](https://github.com/mcbobke/Powershell-Environment/blob/master/scripts/setuphelpers/Install-WinDbg.ps1). In the following chunk of code from that script, I scrape the link to download the setup executable from the [Windows 10 SDK webpage](https://developer.microsoft.com/en-US/windows/downloads/windows-10-sdk):

```powershell
# Get the download link
$url = 'https://developer.microsoft.com/en-US/windows/downloads/windows-10-sdk'
try {
  $response = Invoke-WebRequest -Uri $url -ErrorAction "Stop"
}
catch {
  $PSCmdlet.ThrowTerminatingError($PSItem)
}
$downloadUrl = $response `
| Select-Object -ExpandProperty "Links" `
| Where-Object {$PSItem.innerText -eq 'Download the .EXE'} `
| Select-Object -ExpandProperty "href"
$downloadUrl = 'https:' + $downloadUrl
```

On the latest version of Windows PowerShell (5.1 as of writing), filtering the `Links` works just fine. When testing this script on Core (version 6.0+) for the first time, I discovered the `innerText` property doesn't exist and I got a shell full of errors. _Super_.

As it turns out, things can change between versions of software. Who knew? Mark Kraus has a great [blog post](https://get-powershellblog.blogspot.com/2017/11/powershell-core-web-cmdlets-in-depth.html#L07) about his work on the web cmdlet API change from `System.Net.WebRequest` to `System.Net.Http.HttpClient` in PowerShell Core. After a bit of `Get-Member`-ing around with `Invoke-WebRequest`, I was able to determine this fix:

```powershell
# No longer working on PS 6.0
<# $downloadUrl = $response `
| Select-Object -ExpandProperty "Links" `
| Where-Object {$PSItem.innerText -eq 'Download the .EXE'} `
| Select-Object -ExpandProperty "href" #>

# PS 6.0 Conversion
$downloadUrl = $response `
| Select-Object -ExpandProperty "Links" `
| Where-Object {$PSItem.outerHTML.Contains('Download the .EXE')} `
| Select-Object -ExpandProperty "href"
$downloadUrl = 'https:' + $downloadUrl
```

Of course, this was an intentional difference between Windows PowerShell and PowerShell Core. What about an unintentional difference? Part 2 will come soon, hopefully; as soon as a pull request that I have open is closed!