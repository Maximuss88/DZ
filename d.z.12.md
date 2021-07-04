�������� ����������� � ����� ������������ ������� ���������� ������� ��, ��� ��� �������� �������. ������� � ������ ������� ��������������� ������� ������������ ������� ���� ������� ������ � AWS (Amazon Web Services).
������ 1. ����������� � aws � ���������� � �������� (�������������, �� ������ ����������).
��������� ������� ����� ����� ��������� � ��� ����� ��������, �� � ��� ����� ����� ������� ������ ���� ���������.
AWS ������������� ���������� ����� ���������� �������� � ������ ��� ����� �����������, �������� ������� �����.
    �������� ������ aws.
    ���������� c aws-cli https://aws.amazon.com/cli/.
    ��������� ��������� ��������� aws-sli https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html.
    �������� IAM �������� ��� ���������� c �������
        AmazonEC2FullAccess
        AmazonS3FullAccess
        AmazonDynamoDBFullAccess
        AmazonRDSFullAccess
        CloudWatchFullAccess
        IAMFullAccess
    �������� ���������� ���������
    export AWS_ACCESS_KEY_ID=(your access key id)
    export AWS_SECRET_ACCESS_KEY=(your secret access key)
    ��������, ���������� � ������� ec2 ������� (����� � �������� free tier) ����� ��� ���������.
� ���� ���������� ������� ��������� ����� ������� aws configure list.


[max@max_centos AWS]$ pip3 install awscli
........
Successfully installed awscli-1.19.97 botocore-1.20.97 colorama-0.4.3 docutils-0.15.2 jmespath-0.10.0 pyasn1-0.4.8 python-dateutil-2.8.1 rsa-4.7.2 s3transfer-0.4.2 six-1.16.0 urllib3-1.26.5

[max@max_centos AWS]$ aws --version
aws-cli/1.19.97 Python/3.6.8 Linux/3.10.0-1160.24.1.el7.x86_64 botocore/1.20.97

������ ������������ ����� ���-���������, �������� �������� ������� �������, � ������� aws configure �������� Access Key ID � Secret Access Key:
    
[max@max_centos AWS]$ aws configure list
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************MPGV shared-credentials-file    
secret_key     ****************Fpq4 shared-credentials-file    
    region                     ru-1      config-file    ~/.aws/config
    
������������� ��������, ����� ���� ��� ��������� � ������ ������� env:
export AWS_ACCESS_KEY_ID=***...
export AWS_SECRET_ACCESS_KEY=***...

[max@max_centos AWS]$ aws configure list
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************MPGV              env    
secret_key     ****************Fpq4              env    
    region                     ru-1      config-file    ~/.aws/config
    
������� � ������ EC2, ������� ������ �� "Red Hat Enterprise Linux 8 (HVM), SSD Volume Type - ami-0ba62214afa52bec7 (64-bit x86)", ��������, ��������� (stop) � �������� (terminate instance), �� �� �����, ��� ��������� ������� �� ������ Instances.

**********
������ 2. ��������� ec2 ����� ���������.
    � �������� terraform ������ ��������� �����������, ������� ��� ������ � ������ �����, �������� ���� main.tf � versions.tf.
    ��������������� ��������� ��� aws. � ���� main.tf �������� ���� provider, � � versions.tf ���� terraform � ��������� ������ required_providers. ������� ����� ��������� ���� ������ ������ ����� provider.
    ��������! � ��� ����������� ������ ������ ���� ������ ����� ������� � ��������. ������� � ���������� ������� �� ��������� �� � ���� ���������� ���������.
    � ����� main.tf �������������� ������ data "aws_ami ��� ������ ami ������ ���������� Ubuntu.
    � ����� main.tf �������� ������� ec2 instance. ������������ ������� ��� ����� ������ ���������� ��� ��� �����������. ����������� ����� ���������� ������ � ������ ����� Example Usage, �� ����������, ������� ������� ���������� ����������.
    �������� data-����� aws_caller_identity � aws_region.
    � ���� outputs.tf ��������� ����� output � ������� �� ������������ � ������ ������:
        AWS account ID,
        AWS user ID,
        AWS ������, ������� ������������ � ������ ������,
        ��������� IP ec2 ��������,
        ������������� ������� � ������� ������ �������.
    ���� �� ��������� ������ �����, �� ��������� ����, ��� �� ������� terraform plan ����������� ��� ������.
� �������� ���������� ������� ������������:
    ����� �� ������: ��� ������ ������ ����������� (�� ����������� �� ������� �������) ����� ������� ���� ����� ami? 
(Amazon Elastic Container Service)
    ������ �� ����������� � �������� ������������� ����������.
https://github.com/Maximuss88/devops-netology/commit/c7e594b03e0d06b297ea0a308d2a0ae5734a6e15

    
  
�������������� ���������:
[max@max_centos terraform]$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 3.0"...
- Installing hashicorp/aws v3.48.0...
- Installed hashicorp/aws v3.48.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!


��������� ������� ���������:
[max@max_centos terraform]$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create
Terraform will perform the following actions:

  # aws_instance.web will be created
  + resource "aws_instance" "web" {
      + ami                                  = "ami-0b29b6e62f2343b46"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = (known after apply)
      + availability_zone                    = (known after apply)
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + id                                   = (known after apply)
      .........
      .........
      .........
Plan: 1 to add, 0 to change, 0 to destroy.