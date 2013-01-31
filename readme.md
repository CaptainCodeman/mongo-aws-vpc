# MongoDB AWS VPC

## Description

A set of cloud formation templates for Amazon Web Services to automatically create a Virtual Private Cloud comprising:

* Public subnet for application instances such as web servers
* Primary private subnet in same availability zone as public subnet for main database instance
* Secondary private subnet in alternate availability zone for backup database instance
* Network ACLs and Securioty Groups to secure traffic to and between subnets and instances
* MongoDB Replicate Set Master instance with 4 x EBS volumes as RAID-10 logical volume
* MongoDB Replicate Set Slave instance with single EBS volume for snapshot backups
* NAT instance to provide external access to private subnet inctances
* Internal DNS service using DnsMasq based off the instance host names
* OpenVPN for VPN access to private instances
* Mongo Monitoring Service (MMS)

NOTE: NAT / DNS / VPN / MMS are all combined in a single t1.micro instance.

## Usage

While the templates can be used as-is they are really intended to be a starting point and customized
for your specific requirements. In particular, you should not use the public templates from my S3 bucket
even though they will work because I could change them at any time and you should be in control of your
own templates in case you want to update the configuration of your cloud formation stack.

So, download the templates, modify the configuration as required and upload and run them from your own
Amazon S3 bucket. The bucket name will be combined with the amazon region selected to create the VPC
for the path to the templates, e.g.

    https://intesoft-cloudformation.s3.amazonaws.com/us-east-1

Instead of a single monolithic template, I've tried to make things modular to allow re-use and re-purposing.
For example, an additional private subnet can easily be added in another availability zone containing
MongoDB Replica Set Slave instances for automatica failover by re-using the subnet-private.template
within the vpc.template and the mongo-slave.template within the stack.template.

### Parameters

[TODO]

As well as the public input parameters for the 'should change' options, there are also some Mappings within
the stack.template and vpc.template to define parameters used by the MongoDB instances and the VPC networks.

I would especially recommend changing the instance types to t1.micro while experimenting with the templates
so that you do not incur unecessary AWS charges.

## Notes

I'm not a linux expert by any stretch and this is my first attempt at bash scripting, setting up linux
or cloud formation templates. Most of the settings are based on best practices from research I've done.
So, if anything isn't quite as it should be - please let me know so I can correct it. If anything is
working particularly well then you probably have other people to thank than me for it!

### OpenVPN

OpenVPN Access Server is installed on the NAT instance to provide secure access to the private VPC subnets.

This is an unlicenses trial which provides 2 connections to use. It can be upgraded if required but I am
intending to replace this with the community edition at some point.

### DNS

I've considered using Amazon Route 53 for the DNS but ended up settling on dnsmasq for the internal DNS.
This uses the Name tag of the instance to provide consistent DNS access internally and, if you set the
DNS server option in the openvpn admin web site, you can also refer to the instances by name when your VPN
connection is active.