# Packer Example: Ubuntu 16.04 with Custom NGINX, Built from Source
This repo contains two packer template files which install the latest mainline release of NGINX from source on the most recent, official Ubuntu 16.04 AMI from Canonical. Provisioning is performed with shell scripts, and the NGINX configuration options can be modified to produce a custom install with any combination of builtin or third-party modules enabled.

[Click here for a complete guide to using these packer templetes.](https://alunablog.com/2018/03/30/packer-template-aws-ec2-ubuntu-nginx/)

## Requirements
You need to have Packer installed on your system to use these examples. Please follow the simple instructions at the link below:

[Install Packer](https://www.packer.io/intro/getting-started/install.html)

You must also have an active AWS account. These templates launch `t2.micro` instances which are included with the free-tier.

## Usage
After installing Packer, clone this repo to your local system using the command below:

`$ | git clone https://github.com/a-luna/packer-examples.git`

You must decide how you are going to provide your AWS access keys to packer. [Read this page](https://www.packer.io/docs/builders/amazon.html#authentication) for more info. I recommend creating a credentials file, the default location Packer checks for this file is **$HOME/.aws/credentials** on Linux and macOS, or **%USERPROFILE%.aws\credentials** on Windows. To accociate your access keys with the default profile, include the lines below in your `credentials` file:

```
[default]
aws_access_key_id = YOUR ACCESS KEY
aws_secret_access_key = YOUR SECRET KEY

[default]
region = us-west-1
```

The `region` value should match the VPC that you wish to launch the EC2 instance and where the resulting AMI will be stored. The SDK checks the `AWS_PROFILE` environment variable to determine which profile to use. If no `AWS_PROFILE` variable is set, the SDK uses the default profile.

You can optionally specify a different location for Packer to look for the credentials file by setting the environment variable `AWS_SHARED_CREDENTIALS_FILE`. See [Amazon's documentation on specifying profiles](https://docs.aws.amazon.com/sdk-for-go/v1/developer-guide/configuring-sdk.html#specifying-profiles) for more details.

### nginx_ubuntu_from_source.json
You must build this packer template first, since the other template, `nginx_ubuntu_from_deb.json`, installs NGINX from a .deb file which is created from this template.

If your account has a default VPC setup, you can remove the three lines below from the template file (lines 32-34) and packer will launch the EC2 instance within your default VPC. If you do not have a default VPC setup or you wish to launch the EC2 instance in a different VPC, you must provide values for `vpc_id` and `subnet_id`  before you can use the template:

```JSON
"vpc_id": "vpc-xxxxxxxx",
"subnet_id": "subnet-xxxxxxxx",
"associate_public_ip_address": "true"
```

If using a non-default VPC, public IP addresses are not provided by default. If `associate_public_ip_address` is set to `true`, your new instance will get a Public IP.

Make sure Packer is installed and navigate to the directory containing the packer template files. Validate your changes to the template file by running `packer validate`. If the template is not valid, any errors will be output to the console. The output should look similar to below if your changes were made correctly:

```
packer-examples $ | packer validate nginx_ubuntu_from_deb.json
                  | Template validated successfully.
```

To build the AMI, run:

```
packer-examples $ | packer build nginx_ubuntu_from_deb.json
```

You will see a lot of output to the console while packer is creating the AMI, please [see this article for full details of the build automation process](https://alunablog.com/2018/03/30/packer-template-aws-ec2-ubuntu-nginx/).

After a few minutes, Packer should tell you that an AMI with your customized NGINX installation was generated successfully:

```
==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs: AMIs were created:
us-west-1: ami-xxxxxxxx
```

You can launch an instance of this AMI from the AWS Console. NGINX will be configured and running, you can verify by running the command below:

```
$ | sudo systemctl status nginx.service
  | ● nginx.service - A high performance web server and a reverse proxy server
  |    Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: enabled)
  |    Active: active (running) since Sun 2018-04-29 07:55:18 UTC; 18s ago
  |   Process: 1164 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
  |   Process: 1124 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
  |  Main PID: 1171 (nginx)
  |     Tasks: 2
  |    Memory: 4.7M
  |       CPU: 9ms
  |    CGroup: /system.slice/nginx.service
  |            ├─1171 nginx: master process /usr/sbin/nginx -g daemon on; master_process on
  |            └─1174 nginx: worker process
  |
  | Apr 29 07:55:18 ip-172-31-12-194 systemd[1]: Starting A high performance web server and a reverse proxy server...
  | Apr 29 07:55:18 ip-172-31-12-194 systemd[1]: Started A high performance web server and a reverse proxy server.
```
