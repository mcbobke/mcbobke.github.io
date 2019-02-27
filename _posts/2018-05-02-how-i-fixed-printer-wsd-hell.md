---
title: How I Fixed Printer WSD Hell
date: 2018-05-02T17:15:24+00:00
author: Matt Bobke
permalink: /2018/05/02/how-i-fixed-printer-wsd-hell/
categories:
  - PowerShell
tags:
  - ports
  - powershell
  - printerlogic
  - printers
  - wsd
---
One of the most interesting projects that I've gotten to work on has been the migration from a Windows print server to [PrinterLogic](https://www.printerlogic.com/). At Blizzard's Irvine campus alone there are over 125 printers, all of different manufacturers and models due to the lack of a standardized purchasing process (until now, at least). With that many printers and drivers being used by a few thousand people, a physical low-powered Server 2008 machine was no longer the ideal option for a print share. The Print Spooler would often crawl away and hide in a corner somewhere, its long lifetime led to completely inconsistent and meaningless naming conventions, and simple changes such as updating printer drivers could bring things to a crawl.

Enter PrinterLogic, a totally awesome, centralized, direct-IP-based printing service that makes printer management so much less [infuriating](https://www.youtube.com/watch?v=N9wsjroVlu8). Once we successfully walked through the [setup documentation](http://docs.printerlogic.com/Content/C_Installation/Install_Printer_Installer.htm), we were ready to begin importing printers from the old print server via PrinterLogic's built-in migration process.

We ran a migration of all printers as an initial first pass to see what printers would throw an error, hoping we wouldn't have to investigate much. As it turns out, there were 57 printers configured with Web Services for Devices (WSD) ports instead of TCP/IP-based ports on the server, and these ports are not compatible with PrinterLogic. (Explanation of the differences between TCP/IP and WSD [here](https://www.urtech.ca/2013/09/solved-what-is-the-difference-between-a-tcpip-printer-port-and-a-wsd-printer-port/).) We're still not entirely sure how these printers ended up with WSD ports, but it probably has something to do with the fact that dozens of people had touched the old server over the years. I was not looking forward to the process of mapping these printers to their IP addresses by hand through Windows Print Management and sheer luck - but then I realized, _why not PowerShell?_

First, I wanted to gather a list of all printers on the server and match them with their assigned ports. Additionally, they should have a `PortName` value that starts with `WSD` and should not have a `DeviceType` of `Print3D`.

```powershell
$printers = Get-Printer -ComputerName printserver
$ports = Get-PrinterPort -ComputerName printserver
$output = @()

foreach ($printer in $printers){
  foreach ($port in $ports){
    if (($printer.PortName -eq $port.Name) -and ($printer.PortName -like "WSD*") -and ($printer.DeviceType -ne "Print3D")){
      $pair = New-Object -TypeName PSObject
      $pair | Add-Member -MemberType NoteProperty -Name PrinterName -Value $printer.Name
      $pair | Add-Member -MemberType NoteProperty -Name PrinterPort -Value $port.Name
      $pair | Add-Member -MemberType NoteProperty -Name PortIPAddress -Value $port.PrinterHostAddress
      $output += ,$pair
    }
  }
}

$output | Sort-Object -property PrinterPort | Export-Csv -Path 'C:\Temp\printer\ports\export.csv' -Delimiter ','
```

As you can see, I wanted to grab the IP of each printer with the `$port.PrinterHostAddress` property, but unfortunately the whole column came back blank in the CSV. Thankfully, simply changing this to `$port.DeviceURL` returned URLs of the form `http://10.130.17.225/StableWSDiscoveryEndpoint/schemas-xmlsoap-org_ws_2005_04_discovery` that contained the IP, which could then be filtered out by hand or by script.

Once I had a CSV with columns for printer names, port names, and IP addresses, I could create a new TCP/IP port for each printer and assign it accordingly:

```powershell
$csv = Import-CSV 'C:\Temp\printer\ports\export.csv'

foreach ($PrinterName in $csv){
  $P = Get-Printer $PrinterName.Name -ComputerName printserver
  Add-PrinterPort -Name "IP_$($PrinterName.Address)" -PrinterHostAddress $PrinterName.Address -ComputerName printserver
  Set-Printer -Name $P.Name -PortName "IP_$($PrinterName.Address)" -ComputerName printserver
}
```

And wa-la! Everything was good. Nearly all of those printers successfully imported after a second attempt, with just a few failing again due to incompatible drivers. I even managed to avoid breaking any of these shared printers that were installed on user machines; 0 noticeable impact is always a win.