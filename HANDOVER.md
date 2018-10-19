# SAS Viya AWS Infrastructure
- [SAS Viya AWS Infrastructure](#sas-viya-aws-infrastructure)
    - [Repository](#repository)
    - [Buckets](#buckets)
    - [VPC](#vpc)
    - [Elastic Load Balancer](#elastic-load-balancer)
        - [TLS certificates](#tls-certificates)
    - [Ansible Controller](#ansible-controller)
        - [Connecting to Visual, Prog, Controller, and Stateful](#connecting-to-visual-prog-controller-and-stateful)
    - [Proxy Settings](#proxy-settings)
    - [Local Development](#local-development)
        - [Tools](#tools)
        - [Authentication](#authentication)
        - [Shortcut scripts and Workflow](#shortcut-scripts-and-workflow)
## Repository

A GitHub repository (fork) of the AWS SAS Viya Quick Start Guide exists at this URL:  ____?

## Buckets

A bucket was created in Sydney region called "sas-viya-installation". Inside the bucket are the modifications to the quickstart which were required for the VFF environment. The bucket contains, among other things:

- CloudFormation template
- scripts and playbooks downloaded by each EC2 instance upon first boot
- scripts written by Felipe to expedite iterations of development (bin/ directory)
- new playbooks and scripts to adapt each instance to VFF environment

## VPC

The `datatestnonprod` VPC was selected for SAS installation. It has no Internet Gateway (IGW) and has no inbound or outbound connection to the internet.

The VPC CIDR block is 10.113.192.0/19. The Ansible controller runs in the Public subnet of a single AZ. The other instances run in the Private subnet of a single AZ. The CIDR blocks are as follows:

| ITEM | CIDR BLOCK |
| --------- | --------------- |
| VPC | 10.113.192.0/19 |
| public A | 10.113.192.0/22 |
| public B | 10.113.196.0/22 |
| private A | 10.113.200.0/22 |
| private B | 10.113.204.0/22 |

## Elastic Load Balancer

The quickstart creates an ELB to front some of the servers. I was not successful in getting this working. The ELB also supports custom TLS certificates and can be updated manually in the ELB console.

### TLS certificates

Self-signed certificates are created automatically by the quickstart. Felipe will provide the .crt file.

## Ansible Controller

Use the private key, and the user `ec2-user` to connect to the Ansible Controller. Felipe will provide a SSH private key.

>  **HOW VERSENT CONNECTS TO EC2 HOSTS**:
>
> I am currently connecting to a jumpbox in the _velocity-cloudops-prod_ aws account, at IP address **10.113.79.250** with a private key, and then connecting the Ansible Controller in the Public subnet of the datatestnonprod VPC in the _velocity-data-nonprod_ account. This is how Versent connects to the various EC2 hosts in the various VFF VPCs.
>
> The steps are:
> 1.  Connect to the bastion host 10.113.76.250 with private key and user `ec2-user`
> 2.  Connect to the Ansible Controller (see AWS Console for private 10.*.*.* address)
> 3.  Connect to any other SAS Viya hosts.
>
> VA IT may decide upon a different way to for the SAS installers to connect to the Ansible Controller.

### Connecting to Visual, Prog, Controller, and Stateful

Once connected to the Ansible Controller, run `ssh {host}`, where `{host}` is one of: `visual`, `prog`, `controller`, `stateful`

## Proxy Settings

The VPC has no Internet Gateway (IGW) and therefore has no inbound, or outbound, internet connection. VFF have an HTTP proxy in the _velocity-cloudops-prod_ account. The access this proxy, the following environment variables are required.

    export http_proxy="http://proxy.cloudopsprod.aws.velocityfrequentflyer.internal:3128"
    export https_proxy=$http_proxy
    export HTTP_PROXY=$http_proxy
    export HTTPS_PROXY=$http_proxy
    export no_proxy=".internal,.sas,.viya.sas,visual,prog,stateful,controller,localhost,169.254.169.254"
    export NO_PROXY=$no_proxy

The proxy settings are provided as parameters to the CloudFormation template. You can find a shell script under `/etc/profile.d/http_proxy.sh`. `/etc/yum.conf` has also been updated with proxy configuration.

## Local Development

The following steps provide an example of local development workflow (code, create stack, check logs, repeat)

### Tools
At minimum you will need:

  - network access from your VPN account to _velocity-cloudops-prod_ VPC
  - a shell
  - `awscli`
  - `saml2aws`
  - `openssh`
  - VA AD credentials
  - VA VPN client
  - VPN authentication credentials (may differ from AD credentials)

### Authentication

Use the venerable [saml2aws](https://github.com/Versent/saml2aws) to log into AWS environment using the Command-line interface. Temporary (1 hour) credentials are stored in `~/.aws/credentials`. You will need to ask VA IT for AD credentials for `saml2aws` to work.

### Shortcut scripts and Workflow

1.  Copy one of the `parameters-ap-southeast-2a.json` or `parameters-ap-southeast-2b.json` files to `parameters.json`.

    - `bin/sync`

      Once authenticated, you sync all changes to the S3 bucket `sas-viya-installation` in `ap-southeast-2` region.


    - `bin/cfn <optional-string>`

      Create the CloudFormation stack with this command. The optional string is appended to the stack name `SAS-Viya`. `-test` will result in the stack name of `SAS-Viya-test`.

    - `bin/logs`

      In the event of an error, you can download the logs from `~ec2-user/deployment-logs` and `/var/log` to a local directory `log`
