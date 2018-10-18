# SAS Viya AWS Infrastructure
- [SAS Viya AWS Infrastructure](#sas-viya-aws-infrastructure)
    - [Repository](#repository)
    - [Buckets](#buckets)
    - [VPC](#vpc)
    - [Elastic Load Balancer](#elastic-load-balancer)
        - [TLS certificates](#tls-certificates)
    - [Ansible Controller](#ansible-controller)
        - [Visual, Prog, Controller, and Stateful](#visual-prog-controller-and-stateful)
    - [Proxy Settings](#proxy-settings)
    - [Local Development](#local-development)
        - [Authentication](#authentication)
        - [Copy files to s3](#copy-files-to-s3)
        - [Copy the parameters](#copy-the-parameters)
        - [Create CloudFormation Stack](#create-cloudformation-stack)
        - [Download log files from server for local analysis](#download-log-files-from-server-for-local-analysis)
## Repository

A GitHub repository (fork) of the AWS SAS Viya Quick Start Guide exists at this URL:  ____?

## Buckets

A bucket was created in Sydney region called "sas-viya-installation". Inside the bucket are the modifications to the quickstart which were required for the VFF environment. The bucket contains, among other things:

- CloudFormation template
- scripts and playbooks downloaded by each EC2 instance upon first boot
- scripts written by Felipe to expedite iterations of development (bin/ directory)
- new playbooks and scripts to adapt each instance to VFF environment

## VPC

The `datatestnonprod` VPC was selected for SAS installation. It's CIDR block is 10.113.192.0/19. The Ansible controller runs in the Public subnet of a single AZ. The other instances run in the Private subnet of a single AZ. The CIDR blocks are as follows:

| ITEM | CIDR BLOCK |
| --------- | --------------- |
| VPC | 10.113.192.0/19 |
| public A | 10.113.192.0/22 |
| public B | 10.113.196.0/22 |
| private A | 10.113.200.0/22 |
| private B | 10.113.204.0/22 |

## Elastic Load Balancer

The quickstart creates an ELB to front some of the servers. I was not successful in getting this working. The ELB also supports custome TLS certificates and can be updated manually in the ELB console.

### TLS certificates

Self-signed certificates are created automatically by the quickstart. Felipe will provide the .crt file.

## Ansible Controller

Please use the private key, and the user "ec2-user" to connect to the Ansible Controller. Felipe will provide a SSH private key.

>  **HOW VERSENT CONNECTS TO EC2 HOSTS**:
>
> I am currently connecting to a jumpbox in the _velocity-cloudops-prod_ aws account, at IP address 10.113.79.250 with a private key, and then connecting the Ansible Controller in the Public subnet of the datatestnonprod VPC in the _velocity-data-nonprod_ account. This is how Versent connects to the various EC2 hosts in the various VFF VPCs. VA IT may decide upon a different way to for the SAS installers to connect to the Ansible Controller.

### Visual, Prog, Controller, and Stateful

Once connected to the Ansible Controller, run `ssh {host}`, where `{host}` is one of: `visual`, `prog`, `controller`, `stateful`

## Proxy Settings

The VPC has no Internet Gateway (IGW) and therefore has not inbound, or outbound, internet connection. VFF have an HTTP proxy in the _velocity-cloudops-prod_ account. The access this proxy, the following environment variables are required.

    export http_proxy="http://proxy.cloudopsprod.aws.velocityfrequentflyer.internal:3128"
    export https_proxy=$http_proxy
    export HTTP_PROXY=$http_proxy
    export HTTPS_PROXY=$http_proxy
    export no_proxy=".internal,.sas,.viya.sas,visual,prog,stateful,controller,localhost,169.254.169.254"
    export NO_PROXY=$no_proxy

The proxy settings are provided as parameters to the CloudFormation template. You can find a shell script under `/etc/profile.d/http_proxy.conf`. `/etc/yum.conf` has also been updated with proxy configuration.

## Local Development

### Authentication

Use the venerable `saml2aws` to log into AWS environment using the Command-line interface. Temporary (1 hour) credentials are stored in `~/.aws/credentials`. You will need to ask VA IT for AD credentials for `saml2aws` to work.

### Copy files to s3
Once authenticated, you sync all changes to the S3 bucket `sas-viya-installation` in `ap-southeast-2` region.

`bin/sync`

### Copy the parameters

Copy one of the `parameters-ap-southeast-2a.json` or `parameters-ap-southeast-2b.json` files to `parameters.json`.

### Create CloudFormation Stack
Create the CloudFormation stack with this command:

`bin/cfn <optional-string>`

The optional string is appended to the stack name `SAS-Viya`. `-test` will result in the stack name of `SAS-Viya-test`.

### Download log files from server for local analysis

In the event of an error, you can download the logs from `~ec2-user/deployment-logs` and `/var/log` to a local directory `log`

`bin/logs`
