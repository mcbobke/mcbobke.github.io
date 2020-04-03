---
title: 'Deploy and Configure an AWS EC2 IIS Web Server with PowerShell and DSC'
date: 2020-04-03
permalink: /2020/04/03/deploy-and-configure-an-aws-ec2-iis-web-server-with-powershell-and-dsc/
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

For some information about how AWS provides a great management layer over DSC on Windows Instances, take a look at these resources:

* [Run compliance enforcement and view compliant and non-compliant instances using AWS Systems Manager and PowerShell DSC](https://aws.amazon.com/blogs/mt/run-compliance-enforcement-and-view-compliant-and-non-compliant-instances-using-aws-systems-manager-and-powershell-dsc/)
* [Creating Associations that Run MOF Files](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-state-manager-using-mof-file.html)
* [Samples for deploying an AWS Systems Manager Association using the 'AWS-ApplyDSCMofs' Document](https://gist.github.com/austoonz/14ad194db6e55dcee96bf97ea07adb45)

__Note__: If you are using AWS free tier, there may be a _very_ small fee (a few US cents) as a result of running through these commands as you read. I stuck as close to free tier as possible but I believe Systems Manager ends up charging for its execution time which is minimal.

__Another Note__: Alongside the script that actually deploys and configures AWS resources, there is additionally the `AWSPowerShellDemoConfig.ps1` DSC Configuration. I will be mentioning the AWS-specific parts of this Configuration in this post but I'll avoid diving into DSC fundamentals. You would be better suited to read through [Microsoft's documentation](https://docs.microsoft.com/en-us/powershell/scripting/dsc/overview/overview?view=powershell-6) on DSC to get a grasp on general DSC usage.

__Yet Another Note__: This is not the best way to provision infrastructure in an AWS account. There are better tools for the job such as CloudFormation or Terraform. This is purely for experimentation and example!

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

## Create a VPC and Subnet

We'll be using the `10.0.0.0/16` CIDR block for the VPC and deploying the server into a `10.0.0.0/24` subnet.

```powershell
$vpc = New-EC2Vpc -CidrBlock '10.0.0.0/16'
$subnet = New-EC2Subnet -CidrBlock "10.0.0.0/24" -VpcId $vpc.VpcID
```

## Create the Internet Gateway and Configure Route

We'll be using a simple route that just sends all traffic destined for `0.0.0.0/0` (any IP) to the Internet Gateway that we previously created. The default Network Access Control List for the VPC allows all inbound and outbound traffic so no adjustments need to be made there.

```powershell
$internetGateway = New-EC2InternetGateway
$null = Add-EC2InternetGateway -InternetGatewayId $internetGateway.InternetGatewayId -VpcId $vpc.VpcId

$routeTable = Get-EC2RouteTable | Where-Object { $_.VpcId -eq $vpc.VpcId } | Select-Object -First 1

# Route all packets that are destined for 0.0.0.0/0 (the internet/not the local subnet) to the Internet Gateway
$routeArgs = @{
    RouteTableId         = $routeTable.RouteTableId
    GatewayId            = $internetGateway.InternetGatewayId
    DestinationCidrBlock = "0.0.0.0/0"
}
$route = New-EC2Route @routeArgs
```

## Configure Default Security Group

For the purposes of this demonstration, the Security Group that we will be using for our instance will allow HTTP (port 80), RDP (port 3389), and ICMP ping request/reply from any IP. __DO NOT__ ever do this for anything other than experimentation.

```powershell
$securityGroup = Get-EC2SecurityGroup | Where-Object { $_.GroupName -eq 'default' } | Select-Object -First 1

# Allow HTTP in to EC2 instances with this subnet
$httpIpPermission = @{
    IpProtocol = 'tcp'
    FromPort   = '80'
    ToPort     = '80'
    IpRanges   = '0.0.0.0/0'
}

# Allow RDP in to EC2 instances with this subnet (DO NOT EVER DO THIS EXCEPT FOR TESTING)
$rdpIpPermission = @{
    IpProtocol = 'tcp'
    FromPort   = '3389'
    ToPort     = '3389'
    IpRanges   = '0.0.0.0/0'
}

# Allow ping replies from EC2 instances with this subnet
$icmpReplyPermission = @{
    IpProtocol = 'icmp'
    FromPort   = '0' # ICMP Type 0 - Echo Reply https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml#icmp-parameters-types
    ToPort     = '0' # ICMP Code 0 - No Code
    IpRanges   = '0.0.0.0/0'
}

# Allow ping requests from EC2 instances with this subnet
$icmpRequestPermission = @{
    IpProtocol = 'icmp'
    FromPort   = '8' # ICMP Type 0 - Echo Request https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml#icmp-parameters-types
    ToPort     = '0' # ICMP Code 0 - No Code
    IpRanges   = '0.0.0.0/0'
}

$sgIngress = Grant-EC2SecurityGroupIngress -GroupId $securityGroup.GroupId -IpPermission @( $httpIpPermission, $rdpIpPermission, $icmpReplyPermission, $icmpRequestPermission )
```

## Create an IAM Instance Role and Instance Profile

We need and Instance Role with an associated Instance Profile that will allow the following:

* Systems Manager functionality
* S3 access
* EC2 access

For the latter two I've granted full access in this role. This can and __should__ be restricted further in real usage.

```powershell
# The following JSON defines a policy document that allows EC2 instances to assume the AWS instance role that we are creating
$trustRelationshipJson = @"
{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "",
        "Effect": "Allow",
        "Principal": {
          "Service": "ec2.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
      }
    ]
  }
"@
$iamRole = New-IAMRole -RoleName 'AWSPowerShellDemo' -AssumeRolePolicyDocument $trustRelationshipJson
Register-IAMRolePolicy -RoleName 'AWSPowerShellDemo' -PolicyArn 'arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore' # Core SSM functionality
Register-IAMRolePolicy -RoleName 'AWSPowerShellDemo' -PolicyArn 'arn:aws:iam::aws:policy/AmazonS3FullAccess' # S3 access for reading MOFs and writing output
Register-IAMRolePolicy -RoleName 'AWSPowerShellDemo' -PolicyArn 'arn:aws:iam::aws:policy/AmazonEC2FullAccess' # Gather instance details

# The instance profile must have the same name as the role
$instanceProfile = New-IAMInstanceProfile -InstanceProfileName 'AWSPowerShellDemo'
Add-IAMRoleToInstanceProfile -InstanceProfileName 'AWSPowerShellDemo' -RoleName 'AWSPowerShellDemo'
```

## Build an Elastic Block Store Volume for the Instance

To build an EBS volume in code with PowerShell we will need to use some .NET classes from the AWS SDK that comes along with the PowerShell module. Specifically we will need `Amazon.EC2.Model.EbsBlockDevice` and `Amazon.EC2.Model.BlockDeviceMapping`. The first defines the volume itself and the second defines where the device will be mapped on the resulting instance using Linux drive naming.

```powershell
# Using Amazon-provided .NET objects from the module's binaries, create an EBS Block Device that will be used as the boot/storage volume for our instance
# These .NET objects are implicitly loaded once a cmdlet from the module is called, or if you explicitly import the module
$ebsVolume = New-Object Amazon.EC2.Model.EbsBlockDevice
$ebsVolume.VolumeSize = 30
$ebsVolume.VolumeType = 'standard'
$ebsVolume.DeleteOnTermination = $true

# Create the drive mount mapping using Linux formatting to define where the previously created SSD volume will be mounted
# The first partition of the first mount point
$ebsVolumeDeviceMapping = New-Object Amazon.EC2.Model.BlockDeviceMapping
$ebsVolumeDeviceMapping.DeviceName = '/dev/sda1'
$ebsVolumeDeviceMapping.Ebs = $ebsVolume
```

## Let's Build the Instance!

To build the instance, we need a couple of things:

* UserData - a script that will be run upon first boot of the operating system. In our case, this script will create the file `C:\AWSPowerShellDemo\index.html` and add the text `<body><h1>Hello World!</h1></body>` to it. Do you see where we're going with this?
* AMI - The Amazon Machine Image that we will be using for our instance. In our case, this will be a basic Windows Server 2019 image.

```powershell
# This PowerShell script will be run as SYSTEM on instance creation
# https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-windows-user-data.html
$userData = @"
<powershell>
New-Item -ItemType Directory -Path C:\ -Name AWSPowerShellDemo
New-Item -ItemType File -Path C:\AWSPowerShellDemo -Name index.html
Add-Content -Value '<body><h1>Hello World!</h1></body>' -Path C:\AWSPowerShellDemo\index.html -Encoding UTF8
</powershell>
"@

# Grab the base Windows Server 2019 machine image from Amazon (I got the ID from the AWS Console)
$ami = Get-EC2Image -ImageId 'ami-05f5b1fdbdbc92ec7'

# The KeyName parameter wants the name of an EC2 Key Pair that you've generated, here it is used to get the Admin password for the machine if you ever want to RDP to it
#       For Linux instances, it is used to connect via SSH with private key auth
# MinCount/MaxCount are for defining how many instances you wish to deploy with these settings - we only want 1
# EncodeUserData tells the AWS cmdlet that your userdata script is in plaintext and needs to be encoded to base64
$instanceArgs = @{
    ImageId              = $ami.ImageId
    KeyName              = 'AWSPowerShellDemo'
    SecurityGroupID      = $securityGroup.GroupId
    InstanceType         = 't2.micro'
    InstanceProfile_Name = 'AWSPowerShellDemo'
    MinCount             = 1
    MaxCount             = 1
    SubnetId             = $subnet.SubnetId
    UserData             = $userData
    EncodeUserData       = $true
    BlockDeviceMapping   = @($ebsVolumeDeviceMapping)
    Region               = 'us-west-1'
    AssociatePublicIp    = $true
}

$newInstanceReservation = New-EC2Instance @instanceArgs
# New-EC2Instance returns a Reservation object with an Instances attribute that contains the details of the one (or more) instances that were launched
$newInstance = $newInstanceReservation.Instances[0]

Get-EC2Instance | Select-Object -ExpandProperty Instances | Select-Object -Property *
```

## Create S3 Buckets

We need four S3 buckets:

* `dsc-mofs` - for storing the DSC MOF file that is generated and used to configure the instance.
* `dsc-reports` - for storing detailed DSC compliance reports.
* `dsc-status` - for storing DSC compliance summaries.
* `dsc-output` - for storing the output of the configuration execution on our instance.

```powershell
$mofBucket = New-S3Bucket -BucketName 'dsc-mofs'
$reportBucket = New-S3Bucket -BucketName 'dsc-reports'
$statusBucket = New-S3Bucket -BucketName 'dsc-status'
$outputBucket = New-S3Bucket -BucketName 'dsc-output'
```

## Store a Secret Value in AWS SSM Parameter Store

The [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) is essentially a key-value store for configuration data and secrets management. For our use case our parameter will be of the type `SecureString` which is encrypted and decrypted using the AWS Key Management Service (KMS). By including the value of the `SecureString` parameter in plaintext within this demo script I'm clearly defeating the purpose, but I'm assuming that you would otherwise upload this secret separately.

```powershell
# Writes an encrypted key-value pair to the Systems Manager parameter store
Write-SSMParameter -Name 'WebsiteName' -Type SecureString -Value 'AWSPowerShellDemo'
```

## Referencing our Secret Value in DSC

Let's take a small detour and take a look at the [example DSC configuration](https://github.com/mcbobke/AWSPowerShellDemo/blob/master/AWSPowerShellDemoConfig.ps1) that accompanies this demo. There are three things important to how this was written to work with the `AWS-ApplyDSCMofs` SSM document:

1. By default, `AWS-ApplyDSCMofs` installs necessary DSC resource modules on your target machines automatically from the PowerShell Gallery. The call to `Import-DscResource` is what triggers this.
2. The target node is `localhost`. With native DSC, targeting a remote host requires specifying the hostname of that machine. With `AWS-ApplyDSCMofs`, the node is always `localhost` because SSM is copying the configuration to the target machine and running it locally.
3. The compiled MOF does not need to be named after the hostname of the machine that it will target. It can be whatever you want! You'll see this in the next section.

So how do we reference our secret in our configuration? See [here](https://github.com/mcbobke/AWSPowerShellDemo/blob/master/AWSPowerShellDemoConfig.ps1#L101):

```powershell
xWebsite 'AWSPowerShellDemoWebsite' {
    Name             = '{tagssm:WebsiteName}' # This is the token that will be replaced with the value of the encrypted string that we put in Parameter Store
    Ensure           = 'Present'
    SiteId           = 1
    PhysicalPath     = 'C:\AWSPowerShellDemo'
    ApplicationPool  = 'AWSPowerShellDemo'
    State            = 'Started'
    PreloadEnabled   = $false
    EnabledProtocols = 'http'
    DefaultPage      = 'index.html'
    BindingInfo      = @(
        MSFT_xWebBindingInformation {
            Protocol  = 'http'
            IPAddress = '*'
            Port      = 80
        }
    )
}
```

Here we're using token substitution as described in the [official AWS documentation](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-state-manager-using-mof-file.html#systems-manager-state-manager-using-mof-file-tokens). `'{tagssm:WebsiteName}'` means "go get the value of the WebsiteName parameter in Parameter Store and insert it here".

The ability to use token substitution and target the generic `localhost` node with `AWS-ApplyDSCMofs` means that we can write generalized DSC configurations that can be reused for any number of nodes that we want. We can also run _multiple_ different configurations against a single machine with the compliance of each separate configuration tracked. Cool!

## Bringing it all Together

We're just a few steps away from a successful configuration. First, let's create an output directory for the compiled MOF:

```powershell
# Create an output directory for the MOF, clean it if it already exists
$desiredOutputPath = '.\output'
if (-not (Test-Path -Path $desiredOutputPath)) {
    New-Item -Path '.' -Name 'output' -ItemType Directory
} else {
    Get-ChildItem -Path $desiredOutputPath | Remove-Item -Force
}
```

Next, let's build it and use some PowerShell magic to rename the compiled MOF to have the same `BaseName` as the script that contains the configuration:

```powershell
# Rename the output MOF to the same base name as the script
$configurationScript = Get-Item -Path '.\AWSPowerShellDemoConfig.ps1'
$fullPathToScript = $configurationScript.FullName
$mofBuildOutput = & $fullPathToScript -OutputDir $desiredOutputPath
Rename-Item -Path $mofBuildOutput.FullName -NewName "$($configurationScript.BaseName).mof"
```

Then upload the MOF to the S3 bucket that we created earlier specifically for MOFs (I'm using a loop to show an example of what this might look like if you build all of your MOFs to the same directory):

```powershell
foreach ($mof in (Get-ChildItem -Path $desiredOutputPath)) {
    Write-S3Object -BucketName $mofBucket.BucketName -File $mof.FullName
}
```

Alright, we're ready to create an SSM association between our EC2 instance and the `AWS-ApplyDSCMofs` document! First let's look at the code which ultimately calls `New-SSMAssociation`:

```powershell
$dscAssociationArgs = @{
    AssociationName               = "AWSPowerShellDemoDSC"
    Target                        = @(
        @{
            Key    = 'InstanceIds'
            Values = @($newInstance.InstanceId)
        }
    )
    Name                          = 'AWS-ApplyDSCMofs'
    Parameter                     = @{
        MofsToApply                 = 's3:us-west-1:dsc-mofs:AWSPowerShellDemoConfig.mof'
        ServicePath                 = 'awsdsc'
        MofOperationMode            = 'Apply'
        ReportBucketName            = 'us-west-1:dsc-reports'
        StatusBucketName            = 'us-west-1:dsc-status'
        ModuleSourceBucketName      = 'NONE'
        AllowPSGalleryModuleSource  = 'True'
        ProxyUri                    = ''
        RebootBehavior              = 'AfterMof'
        UseComputerNameForReporting = 'False'
        EnableVerboseLogging        = 'True'
        EnableDebugLogging          = 'True'
        ComplianceType              = 'Custom:DSC'
        PreRebootScript             = ''
    }
    ScheduleExpression            = 'cron(0/30 * ? * * *)' # Every 30 minutes
    S3Location_OutputS3BucketName = $outputBucket.BucketName
}

$ssmAssociation = New-SSMAssociation @dscAssociationArgs
```

Most of the parameters are well documented [here](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-state-manager-using-mof-file.html#systems-manager-state-manager-using-mof-file-creating) but I wanted to discuss the `Target` parameter specifically. It takes a list of [Target](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/index.html?page=SSM/TSSMTarget.html&tocid=Amazon_SimpleSystemsManagement_Model_Target) objects that are instantiated with the `Key` and `Values` properties. Here we're targeting a list of instance IDs that only contains the ID of our example EC2 instance. The AWS PowerShell reference is pretty slim on details about what to feed into this parameter but you can see better examples of what the CLI version of this looks like [here](https://docs.aws.amazon.com/systems-manager/latest/userguide/send-commands-multiple.html#send-commands-targeting).

## Give it a few minutes, and...

Head over to the EC2 section of the AWS console and grab the public IP of your instance. Navigate to it in your browser and you'll be greeted with the familiar "Hello World!" that we all know and love. You can also view the Association in the SSM console and see it's compliance report, along with the S3 buckets that now contain output from the execution of the configuration and compliance checks. Feel free to RDP into the instance to poke around.

Now let's clean up our AWS account to avoid incurring fees and leaving an instance open via RDP. __BIG NOTE__: If you are using your account for other things, don't just blindly run this! Clean it up by hand and don't delete things you care about.

```powershell
Remove-SSMAssociation -AssociationId $ssmAssociation.AssociationId -Force
Remove-SSMParameter -Name 'WebsiteName' -Force
Get-S3Bucket | Remove-S3Bucket -Force -DeleteBucketContent
Remove-EC2Instance -InstanceId $newInstance.InstanceId -Force
Remove-IAMRoleFromInstanceProfile -InstanceProfileName 'AWSPowerShellDemo' -RoleName 'AWSPowerShellDemo' -Force
Remove-IAMInstanceProfile -InstanceProfileName 'AWSPowerShellDemo' -Force
Unregister-IAMRolePolicy -PolicyArn 'arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore' -RoleName 'AWSPowerShellDemo' -Force
Unregister-IAMRolePolicy -PolicyArn 'arn:aws:iam::aws:policy/AmazonS3FullAccess' -RoleName 'AWSPowerShellDemo' -Force
Unregister-IAMRolePolicy -PolicyArn 'arn:aws:iam::aws:policy/AmazonEC2FullAccess' -RoleName 'AWSPowerShellDemo' -Force
Remove-IAMRole -RoleName 'AWSPowerShellDemo' -Force
```

Make sure to visit the VPC and EC2 section of the AWS console and delete your VPC and security groups, respectfully.

That's all, folks! Thank you for reading and if you have any questions please leave me a comment or hit me up via email or Twitter. I'd love to chat!