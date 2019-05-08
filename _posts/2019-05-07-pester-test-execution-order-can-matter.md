---
title: 'Pester Test Execution Order CAN Matter!'
date: 2019-05-07
permalink: /2019/05/07/pester-test-execution-order-can-matter/
categories:
  - PowerShell
  - Pester
tags:
  - powershell
  - pester
  - testing
toc: false
---
My most recent update to my PowerShell module [PSSpeedTest](https://github.com/mcbobke/PSSpeedTest) was to split its Pester test files into separate files for each individual public and private function (see [this pull request](https://github.com/mcbobke/PSSpeedTest/pull/41)). After all of the tests had been separated as desired, the test for private function `Remove-iPerf3` was throwing a suspicious warning at the end of its [execution](https://ci.appveyor.com/project/MatthewBobke/psspeedtest/builds/24355576#L229) and the test for `Remove-SpeedTestServer` was [failing entirely](https://ci.appveyor.com/project/MatthewBobke/psspeedtest/builds/24355576#L374). Enabling debug on the line in the function indicated by the stack trace showed permissions errors when attempting to access paths such as `C:\ProgramData\chocolatey\lib\iperf3\tools` and `C:\ProgramData\chocolatey\lib\iperf3\tools\chocolateyinstall.ps1`.

After wracking my brain for a few hours on this issue, I turned to the [PowerShell Virtual User Group on Slack](http://slack.poshcode.org/) for assistance and the solution was much simpler than I had expected. (Thanks, @StevePJP for the help!)

A test for the function `Install-SpeedTestServer` is executed prior to those for both `Remove-iPerf3` and `Remove-SpeedTestServer`; this test kicks off `iperf3` as a process that listens for connections. The test for `Remove-iPerf3` naturally attempts to remove the `iperf3` package which fails as it is locked. The test for `Remove-SpeedTestServer` first tries to install `iperf3` but fails since it is already installed and running.

Ensuring that the `iperf3` [process is stopped](https://github.com/mcbobke/PSSpeedTest/pull/41/commits/745fb8d01bc63b352e20c620cbdc3482ba9f7730) before removal allowed the test for `Remove-iPerf` to pass and consequently the test for `Remove-SpeedTestServer` also passed.

My tests are a bit of a mess right now and will be cleaned up. The reminder that test execution order can have profound impacts on one's test/build environment was welcome.