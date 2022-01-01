https://github.com/netology-code/mnt-homeworks/tree/MNT-7/08-ansible-03-yandex  
***Домашнее задание к занятию "08.03 Использование Yandex Cloud"***  
***Подготовка к выполнению***  
    ***Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.***  
    ***Скачайте playbook из репозитория с домашним заданием и перенесите его в свой репозиторий.***  

***Основная часть***  
    ***Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает kibana.***  
    ***При создании tasks рекомендую использовать модули: get_url, template, yum, apt.***  
    ***Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, сгенерировать конфигурацию с параметрами.***  
    ***Приготовьте свой собственный inventory файл prod.yml.***  
    ***Запустите ansible-lint site.yml и исправьте ошибки, если они есть.***  
    ***Попробуйте запустить playbook на этом окружении с флагом --check.***  
    ***Запустите playbook на prod.yml окружении с флагом --diff. Убедитесь, что изменения на системе произведены.***  
    ***Повторно запустите playbook с флагом --diff и убедитесь, что playbook идемпотентен.***  
    ***Проделайте шаги с 1 до 8 для создания ещё одного play, который устанавливает и настраивает filebeat.***  
    ***Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.***  
    ***Готовый playbook выложите в свой репозиторий, в ответ предоставьте ссылку на него.***  

***Как оформить ДЗ?***  
***Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.***  



    Создал в старом репозитории ansible-test новую ветку yandex, стер файлы старого плейбука и скопировал файлы нового.  
    Затем создал в site.yml еще два play для кибаны и файлбита, добавил для них же два шаблона:  
    [max@max_centos ansible-test]$ ls -l ./templates/  
    итого 12  
    -rw-r--r-- 1 max max 224 дек 25 18:25 elasticsearch.yml.j2  
    -rw-r--r-- 1 max max 294 дек 27 01:59 filebeat.yml.j2  
    -rw-r--r-- 1 max max 159 дек 26 01:59 kibana.yml.j2  

    Завел аккаунт в Yandex Cloud, создал через веб-интерфейс три ВМ (скриншот), указал локального пользователя и скопировал открытый ключ из ~/.ssh/id_ed25519.pub, затем подключился к каждой по ssh:  

    [max@max_centos DZ]$ ssh max@51.250.23.168  
    The authenticity of host '51.250.23.168 (51.250.23.168)' can't be established.  
    ECDSA key fingerprint is SHA256:KUsFdPhLTq62+OyDGNYTG3LHQGUQ/o0y7PO5bNA7lQg.  
    ECDSA key fingerprint is MD5:45:54:46:aa:ca:61:60:d2:09:67:fa:ac:31:55:cf:16.  
    Are you sure you want to continue connecting (yes/no)? yes  
    Warning: Permanently added '51.250.23.168' (ECDSA) to the list of known hosts.  

    [max@el-instance ~]$ hostname  
    el-instance.ru-central1.internal  

    Мне показалось трудным и непривычным использовать предложенную альтернативную структуру плейбука, поэтому вернул структуру к классическому виду и отредактировал /inventory/prod.yml таким образом:  
    ---  
    all:  
      hosts:  
        el-instance:  
          ansible_host: 51.250.23.168  
          ansible_connection: ssh  
          ansible_user: max  

        k-instance:  
          ansible_host: 51.250.30.167  
          ansible_connection: ssh  
          ansible_user: max  

        f-instance:  
          ansible_host: 51.250.30.217  
          ansible_connection: ssh  
          ansible_user: max  

    Соответственно изменил названия хостов в site.yml на свои.  

    Закомментировал play для filebeat и запустил ansible-lint (ошибок нет, два предупреждения):  

    [max@max_centos ansible-test]$ ansible-lint site.yml  
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the  
    controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16  
    2020, 16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be  
    removed from ansible-core in version 2.12. Deprecation warnings can be disabled 
     by setting deprecation_warnings=False in ansible.cfg.  
    WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml  
    WARNING  Listing 2 violation(s) that are fatal  
    risky-file-permissions: File permissions unset or incorrect  
    site.yml:22 Task/Handler: Configure Elasticsearch  

    risky-file-permissions: File permissions unset or incorrect  
    site.yml:49 Task/Handler: Configure Kibana  

    You can skip specific rules or tags by adding them to your configuration file:  
    # .ansible-lint  
    warn_list:  # or 'skip_list' to silence them completely  
      - experimental  # all rules tagged as experimental  

    Finished with 0 failure(s), 2 warning(s) on 1 files.  


    И затем запустил playbook с флагом --check:  
    [max@max_centos ansible-test]$ ansible-playbook -i inventory/prod.yml site.yml --check  
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
    16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
    setting deprecation_warnings=False in ansible.cfg.  

    PLAY [Install Elasticsearch] *********************************************************************************************************************************  
    TASK [Gathering Facts] *************************************************************************************************************************************** 
    ok: [el-instance]  

    TASK [Download Elasticsearch's rpm] ************************************************************************************************************************** 
    changed: [el-instance]  

    TASK [Install Elasticsearch] *********************************************************************************************************************************     fatal: [el-instance]: FAILED! => {"changed": false, "msg": "No RPM file matching '/tmp/elasticsearch-7.14.0-x86_64.rpm' found on system", "rc": 127, "results":  ["No RPM file matching '/tmp/elasticsearch-7.14.0-x86_64.rpm' found on system"]}  

    PLAY RECAP ***************************************************************************************************************************************************     el-instance                : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0  


    Видимо, не хватило прав скачать файл или не прошла проверка сертификата, поэтому добавил в плейбук из предыдущего задания  
    mode: 0755  
    timeout: 60  
    force: true  
    validate_certs: false  


    Запустил снова:
    [max@max_centos ansible-test]$ ansible-playbook -i inventory/prod.yml site.yml --check
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
    16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
    setting deprecation_warnings=False in ansible.cfg.

    PLAY [Install Elasticsearch] *********************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************
    ok: [el-instance]

    TASK [Download Elasticsearch's rpm] **************************************************************************************************************************
    changed: [el-instance]

    TASK [Install Elasticsearch] *********************************************************************************************************************************
    changed: [el-instance]

    TASK [Configure Elasticsearch] *******************************************************************************************************************************
    changed: [el-instance]

    RUNNING HANDLER [restart Elasticsearch] **********************************************************************************************************************
    fatal: [el-instance]: FAILED! => {"changed": false, "msg": "Could not find the requested service elasticsearch: host"}

    NO MORE HOSTS LEFT *******************************************************************************************************************************************

    PLAY RECAP ***************************************************************************************************************************************************
    el-instance                : ok=4    changed=3    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0

    Handler выдал ошибку - не нашел сервиса elasticsearch для перезапуска, поскольку он еще не установлен))
    Мы запускали в режиме тестового прогона --check.

    Теперь запустим с флагом --diff (отработал успешно, статус changed - изменения сделаны, только не перезапустил кибану):
    [max@max_centos ansible-test]$ ansible-playbook -i inventory/prod.yml site.yml --diff
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020,      16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
      setting deprecation_warnings=False in ansible.cfg.

    PLAY [Install Elasticsearch] *********************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************
    ok: [el-instance]
    
    TASK [Download Elasticsearch's rpm] **************************************************************************************************************************
    changed: [el-instance]

    TASK [Install Elasticsearch] *********************************************************************************************************************************
    changed: [el-instance]

    TASK [Configure Elasticsearch] *******************************************************************************************************************************
    --- before: /etc/elasticsearch/elasticsearch.yml
    +++ after: /home/max/.ansible/tmp/ansible-local-16336arrcreaq/tmpldq07854/elasticsearch.yml.j2
    @@ -1,82 +1,7 @@
    -# ======================== Elasticsearch Configuration =========================
    -#
    -# NOTE: Elasticsearch comes with reasonable defaults for most settings.
    ................
    очень длинный вывод всего конфига целиком
    ................
    -# Require explicit names when deleting indices:
    -#
    -#action.destructive_requires_name: true
    +network.host: 0.0.0.0
    +discovery.seed_hosts: ["10.129.0.30"]
    +node.name: node-a
    +cluster.initial_master_nodes: 
    +   - node-a

    changed: [el-instance]

    RUNNING HANDLER [restart Elasticsearch] **********************************************************************************************************************
    changed: [el-instance]

    PLAY [Install Kibana] ****************************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************
    ok: [k-instance]

    TASK [Download Kibana's rpm] *********************************************************************************************************************************
    changed: [k-instance]

    TASK [Install Kibana] ****************************************************************************************************************************************
    changed: [k-instance]

    TASK [Configure Kibana] **************************************************************************************************************************************
    ERROR! The requested handler 'restart Kibana' was not found in either the main handlers list nor in the listening handlers list


    Повторный запуск с флагом --diff (везде статус OK - в изменениях нет необходимости):
    [max@max_centos ansible-test]$ ansible-playbook -i inventory/prod.yml site.yml --diff
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
    16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
    setting deprecation_warnings=False in ansible.cfg.

    PLAY [Install Elasticsearch] *********************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************
    ok: [el-instance]

    TASK [Download Elasticsearch's rpm] **************************************************************************************************************************
    ok: [el-instance]
 
    TASK [Install Elasticsearch] *********************************************************************************************************************************
    ok: [el-instance]

    TASK [Configure Elasticsearch] *******************************************************************************************************************************
    ok: [el-instance]

    PLAY [Install Kibana] ****************************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************
    ok: [k-instance]

    TASK [Download Kibana's rpm] *********************************************************************************************************************************
    ok: [k-instance]

    TASK [Install Kibana] ****************************************************************************************************************************************
    ok: [k-instance]

    TASK [Configure Kibana] **************************************************************************************************************************************
    ok: [k-instance]

    PLAY RECAP ***************************************************************************************************************************************************
    el-instance                : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    k-instance                 : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


    Раскомментировал play "Install Filebeat" и запустил (снова очень длинный вывод конфига, сократил его):
    [max@max_centos ansible-test]$ ansible-playbook -i inventory/prod.yml site.yml --diff
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
    16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
    setting deprecation_warnings=False in ansible.cfg.

    PLAY [Install Elasticsearch] *********************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************
    ok: [el-instance]

    TASK [Download Elasticsearch's rpm] **************************************************************************************************************************
    ok: [el-instance]

    TASK [Install Elasticsearch] *********************************************************************************************************************************
    ok: [el-instance]

    TASK [Configure Elasticsearch] *******************************************************************************************************************************
    ok: [el-instance]

    PLAY [Install Kibana] ****************************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************
    ok: [k-instance]

    TASK [Download Kibana's rpm] *********************************************************************************************************************************
    ok: [k-instance]

    TASK [Install Kibana] ****************************************************************************************************************************************
    ok: [k-instance]

    TASK [Configure Kibana] **************************************************************************************************************************************
    ok: [k-instance]

    PLAY [Install Filebeat] **************************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************
    ok: [f-instance]

    TASK [Download Filebeat's rpm] *******************************************************************************************************************************
    changed: [f-instance]

    TASK [Install Filebeat] **************************************************************************************************************************************
    changed: [f-instance]

    TASK [Configure Filebeat] ************************************************************************************************************************************
    --- before: /etc/filebeat/filebeat.yml
    +++ after: /home/max/.ansible/tmp/ansible-local-18287lnxnngoe/tmp1mj5f6nu/filebeat.yml.j2
    @@ -1,270 +1,5 @@
    -###################### Filebeat Configuration Example #########################
    -
    -# This file is an example configuration file highlighting only the most common
    .....................
    .....................
    .....................
    -# This allows to enable 6.7 migration aliases
    -#migration.6_to_7.enabled: true
    -
    +  host: "http://10.129.0.14:5601"
    +filebeat.config.modules.path: ${path.config}/modules.d/*.yml
    \ No newline at end of file

    changed: [f-instance]

    TASK [Set filebeat systemwork] *******************************************************************************************************************************
    changed: [f-instance]

    RUNNING HANDLER [restart filebeat] ***************************************************************************************************************************

    PLAY RECAP ***************************************************************************************************************************************************
    el-instance                : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    f-instance                 : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    k-instance                 : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
 
