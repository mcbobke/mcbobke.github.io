---
title: 'Deploy and Configure an AWS EC2 IIS Web Server with PowerShell and DSC'
date: 2020-01-06
permalink: /2020/01/06/deploy-and-configure-an-aws-ec2-iis-web-server-with-powershell-and-dsc/
categories:
  - PowerShell
  - DSC
  - AWS
tags:
  - PowerShell
  - DSC
  - AWS
  - SSM
  - EC2
---

This past December I gave a talk at the [SoCal PowerShell User Group](https://www.meetup.com/SoCal-PowerShell-User-Group) about using the `AWSPowerShell` module to work with AWS and deploy an IIS web server into a VPC that is configured with AWS Systems Manager and PowerShell DSC. If you watched the recording or attended, you will remember that the scope of the presentation was large. With this blog post, I hope to add some clarity to the presentation. Let's walk through the script block by block and see what's going on. You can find the script that I ran for this presentation [here on GitHub](https://github.com/mcbobke/AWSPowerShellDemo) along with the slide deck. It would also be a good idea to have the [AWS Tools for PowerShell Cmdlet Reference](https://docs.aws.amazon.com/powershell/latest/reference/) open as well.

__Note__:If you are using AWS free tier, there may be a _very_ small fee (a few US cents) as a result of running through these commands as you read. I stuck as close to free tier as possible but I believe Systems Manager ends up charging for its execution time which is minimal.

__Another Note__: Alongside the script that actually deploys and configures AWS resources, there is additionally the `AWSPowerShellDemoConfig.ps1` DSC Configuration. I will be mentioning the AWS-specific parts of this Configuration in this post but I'll avoid diving into DSC fundamentals. You would be better suited to read through [Microsoft's documentation](https://docs.microsoft.com/en-us/powershell/scripting/dsc/overview/overview?view=powershell-6) on DSC to get a grasp on general DSC usage.

## AWS API Credential Setup

For the purposes of this post, I'm assuming that you already have an AWS account that you can use for experimentation. You must have created an AWS user (or use the root user if you like to live dangerously and _absolutely are not using a corporate/work account_) and also must have an API access key and secret key associated with that user.

Lets begin by getting that account's API credentials set up in PowerShell with the `Set-AWSCredential` and `Initialize-AWSDefaults` commands.

```powershell
# Uncomment and run the following cmdlet with your own values to store your AWS API credentials in a local encrypted store
# Set-AWSCredential -StoreAs 'MattBobkeAWS1' -AccessKey 'ACCESSKEY' -SecretKey 'SECRETKEY'

# This block loads your stored credentials and sets a default region for the rest of this script
$AWSProfileName = "MattBobkeAWS1"
try {
    Initialize-AWSDefaults -ProfileName $AWSProfileName -Region 'us-west-1' -ErrorAction 'Stop'
} catch {
    $PSCmdlet.ThrowTerminatingError($_)
}
```

`Set-AWSCredential` will store your API credentials in your user folder under `C:\Users\USERNAME\AppData\Local\AWSToolkit\RegisteredAccounts.json` which will then be available for any future AWS work that you do with PowerShell.

In this instance the credentials are stored under the name `MattBobkeAWS1` which I can reference with `Initialize-AWSDefaults` as above. This command sets default configurations for the AWS command line in the current PowerShell session. Here I am using it to load the API credentials for my IAM user account and set the default region to `us-west-1` so that any resources that I deploy later on get deployed into that region.
