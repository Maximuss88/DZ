Зачастую разбираться в новых инструментах гораздо интересней понимая то, как они работают изнутри. Поэтому в рамках первого необязательного задания предлагается завести свою учетную запись в AWS (Amazon Web Services).
Задача 1. Регистрация в aws и знакомство с основами (необязательно, но крайне желательно).
Остальные задания можно будет выполнять и без этого аккаунта, но с ним можно будет увидеть полный цикл процессов.
AWS предоставляет достаточно много бесплатных ресурсов в первых год после регистрации, подробно описано здесь.
    Создайте аккаут aws.
    Установите c aws-cli https://aws.amazon.com/cli/.
    Выполните первичную настройку aws-sli https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html.
    Создайте IAM политику для терраформа c правами
        AmazonEC2FullAccess
        AmazonS3FullAccess
        AmazonDynamoDBFullAccess
        AmazonRDSFullAccess
        CloudWatchFullAccess
        IAMFullAccess
    Добавьте переменные окружения
    export AWS_ACCESS_KEY_ID=(your access key id)
    export AWS_SECRET_ACCESS_KEY=(your secret access key)
    Создайте, остановите и удалите ec2 инстанс (любой с пометкой free tier) через веб интерфейс.
В виде результата задания приложите вывод команды aws configure list.


[max@max_centos AWS]$ pip3 install awscli
........
Successfully installed awscli-1.19.97 botocore-1.20.97 colorama-0.4.3 docutils-0.15.2 jmespath-0.10.0 pyasn1-0.4.8 python-dateutil-2.8.1 rsa-4.7.2 s3transfer-0.4.2 six-1.16.0 urllib3-1.26.5

[max@max_centos AWS]$ aws --version
aws-cli/1.19.97 Python/3.6.8 Linux/3.10.0-1160.24.1.el7.x86_64 botocore/1.20.97

Создал пользователя через веб-интерфейс, назначил политику полного доступа, с помощью aws configure настроил Access Key ID и Secret Access Key:
    
[max@max_centos AWS]$ aws configure list
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************MPGV shared-credentials-file    
secret_key     ****************Fpq4 shared-credentials-file    
    region                     ru-1      config-file    ~/.aws/config
    
Экспортировал значения, после чего они появились в выводе команды env:
export AWS_ACCESS_KEY_ID=***...
export AWS_SECRET_ACCESS_KEY=***...

[max@max_centos AWS]$ aws configure list
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************MPGV              env    
secret_key     ****************Fpq4              env    
    region                     ru-1      config-file    ~/.aws/config
    
Перешел в раздел EC2, вручную создал ВМ "Red Hat Enterprise Linux 8 (HVM), SSD Volume Type - ami-0ba62214afa52bec7 (64-bit x86)", запустил, остановил (stop) и отключил (terminate instance), но не нашёл, как полностью удалить из списка Instances.

**********
Задача 2. Созданием ec2 через терраформ.
    В каталоге terraform вашего основного репозитория, который был создан в начале курсе, создайте файл main.tf и versions.tf.
    Зарегистрируйте провайдер для aws. В файл main.tf добавьте блок provider, а в versions.tf блок terraform с вложенным блоком required_providers. Укажите любой выбранный вами регион внутри блока provider.
    Внимание! В гит репозиторий нельзя пушить ваши личные ключи доступа к аккаунта. Поэтому в предыдущем задании мы указывали их в виде переменных окружения.
    В файле main.tf воспользуйтесь блоком data "aws_ami для поиска ami образа последнего Ubuntu.
    В файле main.tf создайте рессурс ec2 instance. Постарайтесь указать как можно больше параметров для его определения. Минимальный набор параметров указан в первом блоке Example Usage, но желательно, указать большее количество параметров.
    Добавьте data-блоки aws_caller_identity и aws_region.
    В файл outputs.tf поместить блоки output с данными об используемых в данный момент:
        AWS account ID,
        AWS user ID,
        AWS регион, который используется в данный момент,
        Приватный IP ec2 инстансы,
        Идентификатор подсети в которой создан инстанс.
    Если вы выполнили первый пункт, то добейтесь того, что бы команда terraform plan выполнялась без ошибок.
В качестве результата задания предоставьте:
    Ответ на вопрос: при помощи какого инструмента (из разобранных на прошлом занятии) можно создать свой образ ami? 
(Amazon Elastic Container Service)
    Ссылку на репозиторий с исходной конфигурацией терраформа.
https://github.com/Maximuss88/devops-netology/commit/c7e594b03e0d06b297ea0a308d2a0ae5734a6e15

    
  
Инициализируем терраформ:
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


Проверяем будущие изменения:
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