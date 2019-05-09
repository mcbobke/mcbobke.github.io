---
title: 'Introducing My First PowerShell Module &#8211; PSSpeedTest'
date: 2018-06-05T23:47:20+00:00
permalink: /2018/06/05/introducing-my-first-powershell-module-psspeedtest/
categories:
  - PowerShell
tags:
  - module
  - powershell
  - psgallery
  - psspeedtest
  - windowspowershell
---
Writing a PowerShell module was a goal that I've had for myself since I learned of [PowerShell Gallery](https://www.powershellgallery.com/) and helper modules such as [Plaster](https://github.com/PowerShell/Plaster), [PSDeploy](https://github.com/RamblingCookieMonster/PSDeploy), [BuildHelpers](https://github.com/RamblingCookieMonster/BuildHelpers), [InvokeBuild](https://github.com/nightroman/Invoke-Build), and [Pester](https://github.com/pester/Pester). I have never worked with CI/CD build pipelines prior to this and a tool like [AppVeyor](https://www.appveyor.com/) was extremely foreign. I'm really happy with the progress that I've made, and while it isn't perfect by any means, it works!

## Background

When I decided that I wanted to write a module, I had the same thought that I've had plenty of times in the past when I set out to make a video game: _I have no idea what I actually want to make_. However, I did know that I wanted to try my best to check a few boxes with my first module:

* The module needed to be as user-friendly and small in scale as possible, while still maintaining a realistic use
* The module should be useful in an enterprise environment as well as general public use
* A build pipeline should automate all building, testing, and deployment

I've recently done some work with [iPerf](https://iperf.fr/) to test some internal network connections in the office, and I realized that network bandwidth testing was a simple concept (user-friendly), small in scale, useful in an enterprise environment, and useful to the general public. I found that common public network bandwidth testing tools such as [Speedtest by Ookla](http://www.speedtest.net/) don't have a public API, but iPerf does have a Chocolatey package available. Thus, [PSSpeedTest](https://github.com/mcbobke/PSSpeedTest) was born.

## Features

* **Set-SpeedTestConfig** sets the JSON configuration file that stores default speed test servers/ports for public internet and local network speed tests.
* **Get-SpeedTestConfig** prints the current default configuration to the screen.
* **Invoke-SpeedTest** runs a speed test against 1) the stored public internet server (and optional port) using the **-Internet** switch, 2) the stored local network server (and optional port) using the **-Local** switch, or 3) a specified server (and optional port) using the **-Server** and **-Port** switches.
* **Install-SpeedTestServer** sets up an iPerf speed test server on 1) the local computer if **-ComputerName** is not specified (or a domain-joined network computer if it is), 2) on a port besides the default of 5201 if **-Port** is specified, 3) using credentials other than those of the current user if **-Credential** is specified.

I'm still trying to determine what the best means of documenting each function is; currently each function has detailed comment-based help available with `Get-Help FunctionName -Full`, but [platyPS](https://github.com/PowerShell/platyPS) might be a better and more readily-available option that doesn't require installing the module prior to reading the help. I'd rather not have the full usage of each function detailed in the primary module readme.

## Future Plans

* Better help
* A means of decommissioning a speed test server deployed via `Install-SpeedTestServer`

If you're so inclined, give the module a try and submit some pull requests if you see anything that you want to improve! My next post will detail my process of creating the pipeline that I use for working with this module.