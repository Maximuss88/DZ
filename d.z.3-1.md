https://github.com/netology-code/mnt-homeworks/tree/MNT-7/08-ansible-01-base
Домашнее задание к занятию "08.01 Введение в Ansible"
Подготовка к выполнению
    Установите ansible версии 2.10 или выше.
    Создайте свой собственный публичный репозиторий на github с произвольным именем.
    Скачайте playbook из репозитория с домашним заданием и перенесите его в свой репозиторий.

    
Обновляем установочные утилиты (иначе возникает ошибка "No module named 'setuptools_rust'") и затем устанавливаем Ansible:
[root@max_centos ansible-test]# pip3 install --upgrade setuptools
[root@max_centos ansible-test]# pip3 install --upgrade pip

[max@max_centos ansible-test]$ pip3 install ansible
.......
Successfully installed MarkupSafe-2.0.1 ansible-4.7.0 ansible-core-2.11.6 cffi-1.15.0 cryptography-35.0.0 jinja2-3.0.2 packaging-21.2 pycparser-2.20 pyparsing-2.4.7 resolvelib-0.5.4

[max@max_centos ansible-test]$ ansible --version
[DEPRECATION WARNING]: .......
ansible [core 2.11.6] 
.........
  

Основная часть
1. Попробуйте запустить playbook на окружении из test.yml, зафиксируйте какое значение имеет факт some_fact для указанного хоста при выполнении playbook'a.

[max@max_centos playbook]$ ansible-playbook site.yml -i inventory/test.yml
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.

PLAY [Print os facts] ****************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]

TASK [Print OS] **********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "CentOS"
}

TASK [Print fact] ********************************************************************************************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 

2. Найдите файл с переменными (group_vars) в котором задаётся найденное в первом пункте значение и поменяйте его на 'all default fact'.

Отредактировал файл /home/max/ansible-test/playbook/group_vars/all/examp.yml

[max@max_centos playbook]$ ansible-playbook -i inventory/test.yml site.yml
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.

PLAY [Print os facts] ****************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]

TASK [Print OS] **********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "CentOS"
}

TASK [Print fact] ********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  

3. Воспользуйтесь подготовленным (используется docker) или создайте собственное окружение для проведения дальнейших испытаний.

docker run -d --name centos7_1 pycontribs/centos:7 sleep 600000
docker run -d --name ubuntu_1 pycontribs/ubuntu:latest sleep 600000

[max@max_centos playbook]$ docker ps
CONTAINER ID   IMAGE                      COMMAND          CREATED         STATUS         PORTS     NAMES
6d43adf88ad5   pycontribs/ubuntu:latest   "sleep 600000"   5 seconds ago   Up 3 seconds             ubuntu_1
32b286339191   pycontribs/centos:7        "sleep 600000"   3 minutes ago   Up 3 minutes             centos7_1

Взял примеры контейнеров из лекции, также отредактировал prod.yml, изменив заданные имена контейнеров на мои.

4. Проведите запуск playbook на окружении из prod.yml. Зафиксируйте полученные значения some_fact для каждого из managed host.

[max@max_centos playbook]$ ansible-playbook -i inventory/prod.yml site.yml
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.

PLAY [Print os facts] ****************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host ubuntu_1 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See https://docs.ansible.com/ansible-
core/2.11/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be 
disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [ubuntu_1]
ok: [centos7_1]

TASK [Print OS] **********************************************************************************************************************************************
ok: [centos7_1] => {
    "msg": "CentOS"
}
ok: [ubuntu_1] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ********************************************************************************************************************************************
ok: [centos7_1] => {
    "msg": "el"
}
ok: [ubuntu_1] => {
    "msg": "deb"
}

PLAY RECAP ***************************************************************************************************************************************************
centos7_1                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu_1                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

5. Добавьте факты в group_vars каждой из групп хостов так, чтобы для some_fact получились следующие значения: для deb - 'deb default fact', для el - 'el default fact'.

Отредактировал файлы examp.yml

6. Повторите запуск playbook на окружении prod.yml. Убедитесь, что выдаются корректные значения для всех хостов.

[max@max_centos playbook]$ ansible-playbook -i inventory/prod.yml site.yml
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.

PLAY [Print os facts] ****************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host ubuntu_1 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See https://docs.ansible.com/ansible-
core/2.11/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be 
disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [ubuntu_1]
ok: [centos7_1]

TASK [Print OS] **********************************************************************************************************************************************
ok: [centos7_1] => {
    "msg": "CentOS"
}
ok: [ubuntu_1] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ********************************************************************************************************************************************
ok: [centos7_1] => {
    "msg": "el default fact"
}
ok: [ubuntu_1] => {
    "msg": "deb default fact"
}

PLAY RECAP ***************************************************************************************************************************************************
centos7_1                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu_1                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 

7. При помощи ansible-vault зашифруйте факты в group_vars/deb и group_vars/el с паролем netology.

[max@max_centos playbook]$ ansible-vault encrypt ./group_vars/deb/examp.yml
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.
New Vault password: 
Confirm New Vault password: 
Encryption successful

Аналогично /group_vars/el/examp.yml

8. Запустите playbook на окружении prod.yml. При запуске ansible должен запросить у вас пароль. Убедитесь в работоспособности.

[max@max_centos playbook]$ ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.
Vault password: 

PLAY [Print os facts] ****************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host ubuntu_1 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See https://docs.ansible.com/ansible-
core/2.11/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be 
disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [ubuntu_1]
ok: [centos7_1]

TASK [Print OS] **********************************************************************************************************************************************
ok: [ubuntu_1] => {
    "msg": "Ubuntu"
}
ok: [centos7_1] => {
    "msg": "CentOS"
}

TASK [Print fact] ********************************************************************************************************************************************
ok: [centos7_1] => {
    "msg": "el default fact"
}
ok: [ubuntu_1] => {
    "msg": "deb default fact"
}

PLAY RECAP ***************************************************************************************************************************************************
centos7_1                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu_1                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

9. Посмотрите при помощи ansible-doc список плагинов для подключения. Выберите подходящий для работы на control node.

[max@max_centos playbook]$ ansible-doc -t connection --list
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.
[WARNING]: Collection frr.frr does not support Ansible version 2.11.6
[WARNING]: Collection ibm.qradar does not support Ansible version 2.11.6
[WARNING]: Collection splunk.es does not support Ansible version 2.11.6
ansible.netcommon.httpapi      Use httpapi to run command on network appliances                                                                          
ansible.netcommon.libssh       (Tech preview) Run tasks using libssh for ssh connection                                                                  
ansible.netcommon.napalm       Provides persistent connection using NAPALM                                                                               
ansible.netcommon.netconf      Provides a persistent connection using the netconf protocol                                                               
ansible.netcommon.network_cli  Use network_cli to run command on network appliances                                                                      
ansible.netcommon.persistent   Use a persistent unix socket for connection                                                                               
community.aws.aws_ssm          execute via AWS Systems Manager                                                                                           
community.docker.docker        Run tasks in docker containers                                                                                            
community.docker.docker_api    Run tasks in docker containers                                                                                            
community.docker.nsenter       execute on host running controller container                                                                              
community.general.chroot       Interact with local chroot                                                                                                
community.general.funcd        Use funcd to connect to target                                                                                            
community.general.iocage       Run tasks in iocage jails                                                                                                 
community.general.jail         Run tasks in jails                                                                                                        
community.general.lxc          Run tasks in lxc containers via lxc python library                                                                        
community.general.lxd          Run tasks in lxc containers via lxc CLI                                                                                   
community.general.qubes        Interact with an existing QubesOS AppVM                                                                                   
community.general.saltstack    Allow ansible to piggyback on salt minions                                                                                
community.general.zone         Run tasks in a zone instance                                                                                              
community.kubernetes.kubectl   Execute tasks in pods running on Kubernetes                                                                               
community.libvirt.libvirt_lxc  Run tasks in lxc containers via libvirt                                                                                   
community.libvirt.libvirt_qemu Run tasks on libvirt/qemu virtual machines                                                                                
community.okd.oc               Execute tasks in pods running on OpenShift                                                                                
community.vmware.vmware_tools  Execute tasks inside a VM via VMware Tools                                                                                
containers.podman.buildah      Interact with an existing buildah container                                                                               
containers.podman.podman       Interact with an existing podman container                                                                                
kubernetes.core.kubectl        Execute tasks in pods running on Kubernetes                                                                               
local                          execute on controller                                                                                                     
paramiko_ssh                   Run tasks via python ssh (paramiko)                                                                                       
psrp                           Run tasks over Microsoft PowerShell Remoting Protocol                                                                     
ssh                            connect via ssh client binary                                                                                             
winrm                          Run tasks over Microsoft's WinRM

Подходящий для работы на control node - "local (execute on controller)".  

10. В prod.yml добавьте новую группу хостов с именем local, в ней разместите localhost с необходимым типом подключения.

Добавил в ./inventory/prod.yml
  local:
    hosts:
      localhost:
        ansible_connection: local

11. Запустите playbook на окружении prod.yml. При запуске ansible должен запросить у вас пароль. Убедитесь что факты some_fact для каждого из хостов определены из верных group_vars.

[max@max_centos ansible-test]$ ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.
Vault password: 

PLAY [Print os facts] ****************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host ubuntu_1 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See https://docs.ansible.com/ansible-
core/2.11/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be 
disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [ubuntu_1]
ok: [centos7_1]

TASK [Print OS] **********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "CentOS"
}
ok: [centos7_1] => {
    "msg": "CentOS"
}
ok: [ubuntu_1] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [centos7_1] => {
    "msg": "el default fact"
}
ok: [ubuntu_1] => {
    "msg": "deb default fact"
}

PLAY RECAP ***************************************************************************************************************************************************
centos7_1                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu_1                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 

12. Заполните README.md ответами на вопросы. Сделайте git push в ветку master. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым playbook и заполненным README.md.


Необязательная часть
1. При помощи ansible-vault расшифруйте все зашифрованные файлы с переменными.

[max@max_centos ansible-test]$ ansible-vault decrypt ./group_vars/deb/examp.yml
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.
Vault password: 
Decryption successful

2. Зашифруйте отдельное значение PaSSw0rd для переменной some_fact паролем netology. Добавьте полученное значение в group_vars/all/exmp.yml.
3. Запустите playbook, убедитесь, что для нужных хостов применился новый fact.
4. Добавьте новую группу хостов fedora, самостоятельно придумайте для неё переменную. В качестве образа можно использовать этот.

Добавил в prod.yml
  fed:
    hosts:
      fedora_1:
        ansible_connection: docker
        
Также добавил в group_vars /fed/examp.yml

Поднял в докере контейнер fedora_1
[max@max_centos inventory]$ docker ps
CONTAINER ID   IMAGE                      COMMAND          CREATED          STATUS          PORTS     NAMES
b6d31332898c   pycontribs/fedora          "sleep 600000"   14 seconds ago   Up 12 seconds             fedora_1
6d43adf88ad5   pycontribs/ubuntu:latest   "sleep 600000"   3 days ago       Up 3 minutes              ubuntu_1
32b286339191   pycontribs/centos:7        "sleep 600000"   3 days ago       Up 3 minutes              centos7_1

[max@max_centos ansible-test]$ ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.
Vault password: 

PLAY [Print os facts] ****************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]
ok: [fedora_1]
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host ubuntu_1 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See https://docs.ansible.com/ansible-
core/2.11/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be 
disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [ubuntu_1]
ok: [centos7_1]

TASK [Print OS] **********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "CentOS"
}
ok: [centos7_1] => {
    "msg": "CentOS"
}
ok: [ubuntu_1] => {
    "msg": "Ubuntu"
}
ok: [fedora_1] => {
    "msg": "Fedora"
}

TASK [Print fact] ********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [centos7_1] => {
    "msg": "el default fact"
}
ok: [ubuntu_1] => {
    "msg": "deb default fact"
}
ok: [fedora_1] => {
    "msg": "fed default fact"
}

PLAY RECAP ***************************************************************************************************************************************************
centos7_1                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
fedora_1                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu_1                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 

5. Напишите скрипт на bash: автоматизируйте поднятие необходимых контейнеров, запуск ansible-playbook и остановку контейнеров.

#!/bin/bash
docker start b6d31332898c 6d43adf88ad5 32b286339191
ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
docker stop b6d31332898c 6d43adf88ad5 32b286339191

Делаем файл исполняемым (chmod 764 1.sh) и запускаем

[max@max_centos ansible-test]$ ./1.sh
b6d31332898c
6d43adf88ad5
32b286339191
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.
Vault password: 

PLAY [Print os facts] ****************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]
ok: [fedora_1]
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host ubuntu_1 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See https://docs.ansible.com/ansible-
core/2.11/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be 
disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [ubuntu_1]
ok: [centos7_1]

TASK [Print OS] **********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "CentOS"
}
ok: [centos7_1] => {
    "msg": "CentOS"
}
ok: [ubuntu_1] => {
    "msg": "Ubuntu"
}
ok: [fedora_1] => {
    "msg": "Fedora"
}

TASK [Print fact] ********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [centos7_1] => {
    "msg": "el default fact"
}
ok: [ubuntu_1] => {
    "msg": "deb default fact"
}
ok: [fedora_1] => {
    "msg": "fed default fact"
}

PLAY RECAP ***************************************************************************************************************************************************
centos7_1                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
fedora_1                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
ubuntu_1                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

b6d31332898c
6d43adf88ad5
32b286339191

6. Все изменения должны быть зафиксированы и отправлены в вашей личный репозиторий. 
