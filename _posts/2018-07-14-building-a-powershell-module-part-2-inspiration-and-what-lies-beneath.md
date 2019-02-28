---
title: Building a PowerShell Module – Part 2 – Inspiration and What Lies Beneath
date: 2018-07-14T10:13:52+00:00
permalink: /2018/07/14/building-a-powershell-module-part-2-inspiration-and-what-lies-beneath/
categories:
  - PowerShell
tags:
  - iperf
  - module
  - posh
  - powershell
  - scripting
  - speedtest
  - windowspowershell
---
In my last post in this series, [Building a PowerShell Module - Part 1 - Setting up Plaster](https://mattbobke.com/2018/06/19/building-a-powershell-module-part-1-setting-up-plaster/), I detailed the contents and functionality of my Plaster module template. One detail that I should have included at the end is a visual of what the output of Plaster looks like as `Invoke-Plaster` runs with my manifest. Here's that before we begin the next post:

![Invoke-Plaster Output](/images/powershell_2018-06-26_00-14-51.png)

Additionally, here's the resulting folder structure in the destination path:

![Resulting Folder Structure](/images/explorer_2018-06-26_00-15-38.png)

Now that the groundwork for the actual code base is laid out, I'd like to explain the background to how PSSpeedTest came about and why I've made the choices that I've made regarding its architecture.

# Part 2 - The Why and the How

## The Idea Forms

At the office, we were running into the occasional computer that exhibited slow network speeds to internal resources. We weren't sure if it was a bad network interface setting such as the Speed & Duplex advanced property (which we had seen set to `100 Mbps Full Duplex` before), or something as simple as a faulty cable. What we did want to be sure of is that we identified slow network speeds on newly-deployed computers for new hires before they were used for the first time so that the issue could be troubleshooted immediately.

![Network Driver](/images/rundll32_2018-07-10_22-29-35.png)

Part of our new computer verification process requires the deployment technician who set up the computer to note its hostname, IP address, location, and any additional notes in a Cisco Webex Teams chat channel that our Desktop Support team monitors. Once we see one of our assigned new hires' computers has been deployed, we physically evaluate the computer, license software, and confirm that it's ready to go. During the length of time that both technicians are working with the computer, it's easy to miss slow network speeds simply because we don't perform any network-intensive work. Automating a network speed test as well as the gathering of additional information about the computer seemed like a great option!

Now, we have a PowerShell script that is used to verify each new computer setup. It's executed with a batch file and does the following:

  1. Outputs the IP address of each Ethernet/Wi-fi interface
  2. Checks to see if the Speed & Duplex advanced property is set to `Auto Negotiation` and attempts to correct it automatically if it isn't
  3. Tries to copy the iPerf3 executable from a virtual server on our network that is acting as an iPerf3 server; if it isn't accessible, it downloads the executable from iPerf3's website
  4. Runs an iPerf3 speed test against the iPerf3 server and saves the output as JSON, which is then parsed to identify the send and receive speeds of the computer
  5. Compares the speed test results against a threshold speed and saves the results of the comparison
  6. Prompts the deployment technician for the location of the computer, the name of the new hire, and any relevant notes about the computer that Desktop Support should be aware of
  7. Uses Cisco's REST API for Cisco Webex Teams to send a markdown-formatted message containing the hostname, IP addresses, send speed, receive speed, speed test threshold comparison results, location, new hire name, and notes

Great! We have a standardized way of formatting our computer deployment notifications which also performs some checks for us. Seeing this script become a part of our weekly routines got me thinking that other PowerShell users and corporations could benefit from having an easy way to test network speeds on company computers. _How about a PowerShell module?_

## Designing the Module

Although my previously detailed script from work is only used to test local network speeds, I knew that the use case for testing Internet network speeds is there and I wanted to support both situations in the same manner. iPerf3 doesn't differentiate between local network servers and Internet servers, so treating both types of server as separate concepts rested on me. I decided that I would create a way for users of the module to save a JSON configuration file containing the hostname/IP address and port for both a default local-network iPerf3 server and a default Internet-facing iPerf3 server. PowerShell has great JSON support out of the box, JSON is lightweight, and corporations who might use this module can use their config management tools to set a custom configuration file so that `Invoke-SpeedTest -Internet` and `Invoke-SpeedTest -Local` runs against the same servers on any computer across the company.

Then came deciding how to install iPerf3. Thankfully, iPerf3 is available through the [Chocolatey](https://chocolatey.org/packages/iperf3) package manager which can be utilized with the [ChocolateyGet](https://github.com/jianyunt/ChocolateyGet) PackageProvider (details on why the official Chocolatey PackageProvider isn't functional yet are [here](https://github.com/chocolatey/chocolatey-oneget/issues/5)). This would allow me to install the utility through supported PowerShell package management without directly downloading the executable itself, with the slight drawback of needing to install the ChocolateyGet PackageProvider during the first use of the module.

Finally, I wanted to provide an extremely simple way for users to deploy an iPerf3 server on their local network. PowerShell remoting is very powerful (ha-ha) and allows for the calling of locally-available functions on remote computers via `Invoke-Command`. I decided that I would use the functions that I had already come up with to install ChocolateyGet and iPerf3 to do the same to a designated remote machine, and would write two additional functions to set the necessary firewall Allow rules and set up a Scheduled Task to run iPerf3.

At this point, the different features of the module have been realized and written down. Next will come the implementation details of these features; I want to be thorough in my discussion of each script, so I'll only detail one or two at most in each future post in this series.