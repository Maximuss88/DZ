https://github.com/netology-code/mnt-homeworks/tree/MNT-7/08-ansible-02-playbook
Подготовка к выполнению
    Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.
    Скачайте playbook из репозитория с домашним заданием и перенесите его в свой репозиторий.
    Подготовьте хосты в соотвтествии с группами из предподготовленного playbook.
    Скачайте дистрибутив java и положите его в директорию playbook/files/.

Основная часть
    Приготовьте свой собственный inventory файл prod.yml.
    Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает kibana.
    При создании tasks рекомендую использовать модули: get_url, template, unarchive, file.
    Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, сгенерировать конфигурацию с параметрами.
    Запустите ansible-lint site.yml и исправьте ошибки, если они есть.
    Попробуйте запустить playbook на этом окружении с флагом --check.
    Запустите playbook на prod.yml окружении с флагом --diff. Убедитесь, что изменения на системе произведены.
    Повторно запустите playbook с флагом --diff и убедитесь, что playbook идемпотентен.
    Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
    Готовый playbook выложите в свой репозиторий, в ответ предоставьте ссылку на него.

Как оформить ДЗ?
Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.



Создал новую ветку kibana в старом репозитории ansible-test, перенес туда playbook из задания.
Возникли проблемы при выполнении git push - файл jdk-11.0.13_linux-x64_bin.rpm больше 100 Мб и не укладывается в лимит Гитхаба.
Поэтому пришлось убрать его из отслеживания гита и оставить только локально (git rm --cached ./files/jdk-11.0.13_linux-x64_bin.tar.gz). 

Поднял в докере контейнер на основе CentOS, соответственно сконфигурировал файл inventory/prod.yml:
elasticsearch:
  hosts:
    centos_el:
      ansible_connection: docker
      ansible_user: root

Решил установить Кибану в том же контейнере, что и Эластик, и просто создать для нее новый play в плейбуке site.yml
Добавил для нее новые переменные в group_vars/elasticsearch/vars.yml, также добавил новый шаблон templates/kib.sh.j2

Ansible-lint не оказалось по умолчанию в моей версии (ansible [core 2.11.6]), установил отдельно (pip3 install ansible-lint).
И запустил - ошибок не найдено, только предупреждения:

[max@max_centos ansible-test]$ ansible-lint site.yml
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the 
controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 
2020, 16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be 
removed from ansible-core in version 2.12. Deprecation warnings can be disabled
 by setting deprecation_warnings=False in ansible.cfg.
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml
WARNING  Listing 7 violation(s) that are fatal
risky-file-permissions: File permissions unset or incorrect
site.yml:9 Task/Handler: Upload .tar.gz file containing binaries from local storage

risky-file-permissions: File permissions unset or incorrect
site.yml:16 Task/Handler: Ensure installation dir exists

risky-file-permissions: File permissions unset or incorrect
site.yml:32 Task/Handler: Export environment variables

risky-file-permissions: File permissions unset or incorrect
site.yml:53 Task/Handler: Create directrory for Elasticsearch

risky-file-permissions: File permissions unset or incorrect
site.yml:68 Task/Handler: Set environment Elastic

risky-file-permissions: File permissions unset or incorrect
site.yml:89 Task/Handler: Create directrory for Kibana

risky-file-permissions: File permissions unset or incorrect
site.yml:104 Task/Handler: Set environment Kibana

You can skip specific rules or tags by adding them to your configuration file:
# .ansible-lint
warn_list:  # or 'skip_list' to silence them completely
  - experimental  # all rules tagged as experimental

Finished with 0 failure(s), 7 warning(s) on 1 files.


Затем запустил playbook с флагом --check:

[max@max_centos ansible-test]$ ansible-playbook -i inventory/prod.yml site.yml --check
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.

PLAY [Install Java] ******************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [centos_el]

TASK [Set facts for Java 11 vars] ****************************************************************************************************************************
ok: [centos_el]

TASK [Upload .tar.gz file containing binaries from local storage] ********************************************************************************************
ok: [centos_el]

TASK [Ensure installation dir exists] ************************************************************************************************************************
ok: [centos_el]

TASK [Extract java in the installation directory] ************************************************************************************************************
skipping: [centos_el]

TASK [Export environment variables] **************************************************************************************************************************
changed: [centos_el]

PLAY [Install Elasticsearch] *********************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [centos_el]

TASK [Upload tar.gz Elasticsearch from remote URL] ***********************************************************************************************************
changed: [centos_el]

TASK [Create directrory for Elasticsearch] *******************************************************************************************************************
ok: [centos_el]

TASK [Extract Elasticsearch in the installation directory] ***************************************************************************************************
skipping: [centos_el]

TASK [Set environment Elastic] *******************************************************************************************************************************
changed: [centos_el]

PLAY [Install Kibana] ****************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [centos_el]

TASK [Upload tar.gz Kibana from remote URL] ******************************************************************************************************************
changed: [centos_el]

TASK [Create directrory for Kibana] **************************************************************************************************************************
changed: [centos_el]

TASK [Extract Kibana in the installation directory] **********************************************************************************************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: NoneType: None
fatal: [centos_el]: FAILED! => {"changed": false, "msg": "dest '/opt/kibana/7.15.2' must be an existing dir"}

PLAY RECAP ***************************************************************************************************************************************************
centos_el                  : ok=12   changed=5    unreachable=0    failed=1    skipped=2    rescued=0    ignored=0



Плейбук отработал успешно:

[max@max_centos ansible-test]$ ansible-playbook -i inventory/prod.yml site.yml --diff
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.

PLAY [Install Java] ******************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [centos_el]

TASK [Set facts for Java 11 vars] ****************************************************************************************************************************
ok: [centos_el]

TASK [Upload .tar.gz file containing binaries from local storage] ********************************************************************************************
ok: [centos_el]

TASK [Ensure installation dir exists] ************************************************************************************************************************
ok: [centos_el]

TASK [Extract java in the installation directory] ************************************************************************************************************
ok: [centos_el]

TASK [Export environment variables] **************************************************************************************************************************
ok: [centos_el]

PLAY [Install Elasticsearch] *********************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [centos_el]

TASK [Upload tar.gz Elasticsearch from remote URL] ***********************************************************************************************************
ok: [centos_el]

TASK [Create directrory for Elasticsearch] *******************************************************************************************************************
ok: [centos_el]

TASK [Extract Elasticsearch in the installation directory] ***************************************************************************************************
skipping: [centos_el]

TASK [Set environment Elastic] *******************************************************************************************************************************
--- before: /etc/profile.d/elk.sh
+++ after: /home/max/.ansible/tmp/ansible-local-4685pfu0_vf4/tmp3fgfam6x/elk.sh.j2
@@ -1,5 +1,5 @@
 # Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
 #!/usr/bin/env bash
 
-export KIB_HOME=/opt/kibana/7.15.2
-export PATH=$PATH:$KIB_HOME/bin
\ No newline at end of file
+export ES_HOME=/opt/elastic/7.10.1
+export PATH=$PATH:$ES_HOME/bin
\ No newline at end of file

changed: [centos_el]

PLAY [Install Kibana] ****************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [centos_el]

TASK [Upload tar.gz Kibana from remote URL] ******************************************************************************************************************
changed: [centos_el]

TASK [Create directrory for Kibana] **************************************************************************************************************************
ok: [centos_el]

TASK [Extract Kibana in the installation directory] **********************************************************************************************************
skipping: [centos_el]

TASK [Set environment Kibana] ********************************************************************************************************************************
--- before
+++ after: /home/max/.ansible/tmp/ansible-local-4685pfu0_vf4/tmpgop4mtx2/kib.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export KIB_HOME=/opt/kibana/7.15.2
+export PATH=$PATH:$KIB_HOME/bin
\ No newline at end of file

changed: [centos_el]

PLAY RECAP ***************************************************************************************************************************************************
centos_el                  : ok=14   changed=3    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0


И повторно:

[max@max_centos ansible-test]$ ansible-playbook -i inventory/prod.yml site.yml --diff
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.

PLAY [Install Java] ******************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [centos_el]

TASK [Set facts for Java 11 vars] ****************************************************************************************************************************
ok: [centos_el]

TASK [Upload .tar.gz file containing binaries from local storage] ********************************************************************************************
ok: [centos_el]

TASK [Ensure installation dir exists] ************************************************************************************************************************
ok: [centos_el]

TASK [Extract java in the installation directory] ************************************************************************************************************
ok: [centos_el]

TASK [Export environment variables] **************************************************************************************************************************
ok: [centos_el]

PLAY [Install Elasticsearch] *********************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [centos_el]

TASK [Upload tar.gz Elasticsearch from remote URL] ***********************************************************************************************************
ok: [centos_el]

TASK [Create directrory for Elasticsearch] *******************************************************************************************************************
ok: [centos_el]

TASK [Extract Elasticsearch in the installation directory] ***************************************************************************************************
skipping: [centos_el]

TASK [Set environment Elastic] *******************************************************************************************************************************
ok: [centos_el]

PLAY [Install Kibana] ****************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [centos_el]

TASK [Upload tar.gz Kibana from remote URL] ******************************************************************************************************************
ok: [centos_el]

TASK [Create directrory for Kibana] **************************************************************************************************************************
ok: [centos_el]

TASK [Extract Kibana in the installation directory] **********************************************************************************************************
skipping: [centos_el]

TASK [Set environment Kibana] ********************************************************************************************************************************
ok: [centos_el]

PLAY RECAP ***************************************************************************************************************************************************
centos_el                  : ok=14   changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0



Архивы появились и были распакованы, каталоги созданы:

[root@a0a114f2c375 /]# ls -la /tmp
total 546896
drwxrwxrwt 1 root root       121 Dec  7 18:25 .
drwxr-xr-x 1 root root        51 Dec  6 17:52 ..
drwxrwxrwt 2 root root         6 Dec  4  2020 .ICE-unix
drwxrwxrwt 2 root root         6 Dec  4  2020 .Test-unix
drwxrwxrwt 2 root root         6 Dec  4  2020 .X11-unix
drwxrwxrwt 2 root root         6 Dec  4  2020 .XIM-unix
drwxrwxrwt 2 root root         6 Dec  4  2020 .font-unix
-rwxr-xr-x 1 1000 1000 318801277 Dec  6 17:50 elasticsearch-7.10.1-linux-x86_64.tar.gz
-rw-rw-r-- 1 1000 1000 151029085 Dec  6 17:25 jdk-11.0.13.tar.gz
-rwxr-xr-x 1 root root  90176848 Dec  7 18:24 kibana-7.15.2-linux-x86_64.tar.gz
-rwx------ 1 root root       701 Dec  4  2020 ks-script-esd4my7v
-rwx------ 1 root root       671 Dec  4  2020 ks-script-eusq_sc5


[root@a0a114f2c375 /]# ls -la /opt
total 0
drwxr-xr-x 1 root root 46 Dec  7 18:14 .
drwxr-xr-x 1 root root 51 Dec  6 17:52 ..
drwxr-xr-x 3 root root 20 Dec  6 17:46 elastic
drwxr-xr-x 3 root root 21 Dec  6 17:30 jdk
drwxr-xr-x 3 root root 20 Dec  7 18:14 kibana


[root@a0a114f2c375 /]# ls -la /opt/kibana/7.15.2/
total 1496
drwxr-xr-x  10 root root     210 Dec  7 18:15 .
drwxr-xr-x   3 root root      20 Dec  7 18:14 ..
-rw-r--r--   1 root root    3733 Nov  4 13:30 .i18nrc.json
-rw-r--r--   1 root root    3860 Nov  4 13:30 LICENSE.txt
-rw-r--r--   1 root root 1473082 Nov  4 13:30 NOTICE.txt
-rw-r--r--   1 root root    3970 Nov  4 13:30 README.txt
drwxr-xr-x   2 root root      94 Nov  4 13:31 bin
drwxr-xr-x   2 root root      44 Nov  4 13:30 config
drwxr-xr-x   2 root root       6 Nov  4 13:30 data
drwxr-xr-x   6 root root     108 Nov  4 13:30 node
drwxr-xr-x 838 root root   24576 Nov  4 13:30 node_modules
-rw-r--r--   1 root root     740 Nov  4 13:30 package.json
drwxr-xr-x   2 root root       6 Nov  4 13:30 plugins
drwxr-xr-x  10 root root     143 Nov  4 13:30 src
drwxr-xr-x   3 root root      96 Nov  4 13:30 x-pack


[root@a0a114f2c375 /]# ls -la /etc/profile.d/
total 68
drwxr-xr-x 1 root root   48 Dec  8 16:41 .
drwxr-xr-x 1 root root   23 Dec  6 16:28 ..
-rw-r--r-- 1 root root  196 May 11  2019 colorgrep.csh
-rw-r--r-- 1 root root  201 May 11  2019 colorgrep.sh
-rw-r--r-- 1 root root  162 May 11  2019 colorxzgrep.csh
-rw-r--r-- 1 root root  183 May 11  2019 colorxzgrep.sh
-rw-r--r-- 1 root root  216 Nov  8  2019 colorzgrep.csh
-rw-r--r-- 1 root root  220 Nov  8  2019 colorzgrep.sh
-rw-r--r-- 1 root root   80 May 15  2020 csh.local
-rw-r--r-- 1 root root  184 Dec  8 16:39 elk.sh
-rw-r--r-- 1 root root 1107 Dec 14  2017 gawk.csh
-rw-r--r-- 1 root root  757 Dec 14  2017 gawk.sh
-rw-r--r-- 1 root root  185 Dec  7 18:11 jdk.sh
-rw-r--r-- 1 root root  185 Dec  8 16:41 kib.sh
-rw-r--r-- 1 root root 2486 May 15  2020 lang.csh
-rw-r--r-- 1 root root 2312 May 15  2020 lang.sh
-rw-r--r-- 1 root root  500 May 11  2019 less.csh
-rw-r--r-- 1 root root  253 May 11  2019 less.sh
-rw-r--r-- 1 root root   81 May 15  2020 sh.local
 
