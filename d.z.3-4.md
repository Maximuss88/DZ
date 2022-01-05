https://github.com/netology-code/mnt-homeworks/tree/MNT-7/08-ansible-04-role
Подготовка к выполнению
    Создайте два пустых публичных репозитория в любом своём проекте: kibana-role и filebeat-role.
    Добавьте публичную часть своего ключа к своему профилю в github.

Основная часть
Наша основная цель - разбить наш playbook на отдельные roles. Задача: сделать roles для elastic, kibana, filebeat и написать playbook для использования этих ролей. Ожидаемый результат: существуют два ваших репозитория с roles и один репозиторий с playbook.
    Создать в старой версии playbook файл requirements.yml и заполнить его следующим содержимым:
    ---
      - src: git@github.com:netology-code/mnt-homeworks-ansible.git
        scm: git
        version: "2.0.0"
        name: elastic 

    При помощи ansible-galaxy скачать себе эту роль.
    Создать новый каталог с ролью при помощи ansible-galaxy role init kibana-role.
    На основе tasks из старого playbook заполните новую role. Разнесите переменные между vars и default.
    Перенести нужные шаблоны конфигов в templates.
    Создать новый каталог с ролью при помощи ansible-galaxy role init filebeat-role.
    На основе tasks из старого playbook заполните новую role. Разнесите переменные между vars и default.
    Перенести нужные шаблоны конфигов в templates.
    Описать в README.md обе роли и их параметры.
    Выложите все roles в репозитории. Проставьте тэги, используя семантическую нумерацию.
    Добавьте roles в requirements.yml в playbook.
    Переработайте playbook на использование roles.
    Выложите playbook в репозиторий.
    В ответ приведите ссылки на оба репозитория с roles и одну ссылку на репозиторий с playbook.

Как оформить ДЗ?
Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.



Создал два новых репозитория kibana-role и filebeat-role.

И создал в Yandex Cloud новую ВМ r-instance, добавил её в hosts.yml, все изменения будем производить на ней.

В старом репозитории ansible-test создал новую ветку roles и добавил в неё файл requirements.yml, отправил изменения (git push origin roles)

Скачал роль elastic:

[max@max_centos ansible-test]$ ansible-galaxy install -r requirements.yml
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.
Starting galaxy role install process
- extracting elastic to /home/max/.ansible/roles/elastic
- elastic (2.0.0) was installed successfully

Перешел в тот же каталог и создал ещё две пустых роли:

[max@max_centos roles]$ ansible-galaxy role init kibana-role
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.
- Role kibana-role was created successfully
[max@max_centos roles]$ ansible-galaxy role init filebeat-role
........
- Role filebeat-role was created successfully
[max@max_centos roles]$ ls
elastic  filebeat-role  kibana-role

Перенес tasks в новые роли, перекинул шаблоны конфигов в templates.

Выложил новые roles в репозитории (с кибаной были проблемы из-за разных названий веток - локальной master и удалённой main, с файлбитом сделал иначе, сразу переименовал ветку):
cd ../filebeat-role/
git init
git remote add origin git@github.com:Maximuss88/filebeat-role.git
-инициализировал ветку master, сделав коммит-
git branch -m master main
git pull origin main
git push origin main


Добавил новые roles в requirements.yml:
[max@max_centos ansible-test]$ cat requirements.yml 
---
  - src: git@github.com:netology-code/mnt-homeworks-ansible.git
    scm: git
    version: "2.0.0"
    name: elastic
  - src: git@github.com:Maximuss88/kibana-role.git
    scm: git
    name: kibana-role
  - src: git@github.com:Maximuss88/filebeat-role.git
    scm: git
    name: filebeat-role


Переделал playbook на использование roles:
[max@max_centos ansible-test]$ cat site.yml 
---
- name: Install Elasticsearch
  hosts: r-instance
  roles:
    - elastic
    - kibana-role
    - filebeat-role
    

И файл hosts:
[max@max_centos ansible-test]$ cat ./inventory/prod.yml 
---
all:
  hosts:
    r-instance:
      ansible_host: 51.250.22.89
      ansible_connection: ssh
      ansible_user: max
      
      
И наконец, запустил playbook с ролями (раза с пятого-шестого, было несколько проблем, правил конфиги, указывал полный путь вместо относительного, однажды скачался битый архив и т.д.):
[max@max_centos ansible-test]$ ansible-playbook -i inventory/prod.yml site.yml 
[DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.

PLAY [Install Elasticsearch] *********************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [r-instance]

TASK [elastic : Fail if unsupported system detected] *********************************************************************************************************
skipping: [r-instance]

TASK [elastic : include_tasks] *******************************************************************************************************************************
included: /home/max/.ansible/roles/elastic/tasks/download_yum.yml for r-instance

TASK [elastic : Download Elasticsearch's rpm] ****************************************************************************************************************
ok: [r-instance -> localhost]

TASK [elastic : Copy Elasticsearch to managed node] **********************************************************************************************************
ok: [r-instance]

TASK [elastic : include_tasks] *******************************************************************************************************************************
included: /home/max/.ansible/roles/elastic/tasks/install_yum.yml for r-instance

TASK [elastic : Install Elasticsearch] ***********************************************************************************************************************
ok: [r-instance]

TASK [elastic : Configure Elasticsearch] *********************************************************************************************************************
ok: [r-instance]

TASK [kibana-role : Fail if unsupported system detected] *****************************************************************************************************
skipping: [r-instance]

TASK [kibana-role : include_tasks] ***************************************************************************************************************************
included: /home/max/.ansible/roles/kibana-role/tasks/download_yum.yml for r-instance

TASK [kibana-role : Download Kibana's rpm] *******************************************************************************************************************
ok: [r-instance -> localhost]

TASK [kibana-role : Copy Kibana to managed node] *************************************************************************************************************
ok: [r-instance]

TASK [kibana-role : include_tasks] ***************************************************************************************************************************
included: /home/max/.ansible/roles/kibana-role/tasks/install_yum.yml for r-instance

TASK [kibana-role : Install Kibana] **************************************************************************************************************************
ok: [r-instance]

TASK [kibana-role : Configure Kibana] ************************************************************************************************************************
ok: [r-instance]

TASK [filebeat-role : Fail if unsupported system detected] ***************************************************************************************************
skipping: [r-instance]

TASK [filebeat-role : include_tasks] *************************************************************************************************************************
included: /home/max/.ansible/roles/filebeat-role/tasks/download_yum.yml for r-instance

TASK [filebeat-role : Download Filebeat's rpm] ***************************************************************************************************************
ok: [r-instance -> localhost]

TASK [filebeat-role : Copy Filebeat to managed node] *********************************************************************************************************
ok: [r-instance]

TASK [filebeat-role : include_tasks] *************************************************************************************************************************
included: /home/max/.ansible/roles/filebeat-role/tasks/install_yum.yml for r-instance

TASK [filebeat-role : Install Filebeat] **********************************************************************************************************************
ok: [r-instance]

TASK [filebeat-role : Configure Filebeat] ********************************************************************************************************************
changed: [r-instance]

RUNNING HANDLER [filebeat-role : restart Filebeat] ***********************************************************************************************************
changed: [r-instance]

PLAY RECAP ***************************************************************************************************************************************************
r-instance                 : ok=20   changed=2    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0

