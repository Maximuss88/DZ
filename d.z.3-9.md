https://github.com/netology-code/mnt-homeworks/tree/MNT-7/09-ci-03-cicd

***Домашнее задание к занятию "09.03 CI\CD"***  
***Подготовка к выполнению***  

    Создаём 2 VM в yandex cloud со следующими параметрами: 2CPU 4RAM Centos7(остальное по минимальным требованиям)
    Прописываем в inventory playbook'a созданные хосты
    Добавляем в files файл со своим публичным ключом (id_rsa.pub). Если ключ называется иначе - найдите таску в плейбуке, которая использует id_rsa.pub имя и исправьте на своё
    Запускаем playbook, ожидаем успешного завершения
    Проверяем готовность Sonarqube через браузер
    Заходим под admin\admin, меняем пароль на свой
    Проверяем готовность Nexus через бразуер
    Подключаемся под admin\admin123, меняем пароль, сохраняем анонимный доступ

***Знакомоство с SonarQube***  
***Основная часть***  

    Создаём новый проект, название произвольное
    Скачиваем пакет sonar-scanner, который нам предлагает скачать сам sonarqube
    Делаем так, чтобы binary был доступен через вызов в shell (или меняем переменную PATH или любой другой удобный вам способ)
    Проверяем sonar-scanner --version
    Запускаем анализатор против кода из директории example с дополнительным ключом -Dsonar.coverage.exclusions=fail.py
    Смотрим результат в интерфейсе
    Исправляем ошибки, которые он выявил(включая warnings)
    Запускаем анализатор повторно - проверяем, что QG пройдены успешно
    Делаем скриншот успешного прохождения анализа, прикладываем к решению ДЗ

***Знакомство с Nexus***  
***Основная часть***  

    В репозиторий maven-public загружаем артефакт с GAV параметрами:
        groupId: netology
        artifactId: java
        version: 8_282
        classifier: distrib
        type: tar.gz
    В него же загружаем такой же артефакт, но с version: 8_102
    Проверяем, что все файлы загрузились успешно
    В ответе присылаем файл maven-metadata.xml для этого артефекта

***Знакомство с Maven***  
***Подготовка к выполнению***  

    Скачиваем дистрибутив с maven
    Разархивируем, делаем так, чтобы binary был доступен через вызов в shell (или меняем переменную PATH или любой другой удобный вам способ)
    Удаляем из apache-maven-<version>/conf/settings.xml упоминание о правиле, отвергающем http соединение( раздел mirrors->id: my-repository-http-unblocker)
    Проверяем mvn --version
    Забираем директорию mvn с pom

***Основная часть***  

    Меняем в pom.xml блок с зависимостями под наш артефакт из первого пункта задания для Nexus (java с версией 8_282)
    Запускаем команду mvn package в директории с pom.xml, ожидаем успешного окончания
    Проверяем директорию ~/.m2/repository/, находим наш артефакт
    В ответе присылаем исправленный файл pom.xml

***Как оформить ДЗ?***  
***Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.***  




Создал 2 ВМ в Yandex Cloud, подключился по разу к каждой, добавил их IP в inventory/cicd/hosts.yml, скопировал в каталог files файл id_ed25519.pub, исправил имя файла в плейбуке.

    Плейбук успешно отработал:
    [max@max_centos infrastructure]$ ansible-playbook -i inventory/cicd/hosts.yml site.yml 
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.6.8 (default, Nov 16 2020, 
    16:55:22) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by 
    setting deprecation_warnings=False in ansible.cfg.

    PLAY [Get OpenJDK installed] *********************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************
    ok: [sonar-01]

    TASK [install unzip] *****************************************************************************************************************************************
    changed: [sonar-01]

    TASK [Upload .tar.gz file conaining binaries from remote storage] ********************************************************************************************
    changed: [sonar-01]

    TASK [Ensure installation dir exists] ************************************************************************************************************************
    changed: [sonar-01]

    TASK [Extract java in the installation directory] ************************************************************************************************************
    changed: [sonar-01]

    TASK [Export environment variables] **************************************************************************************************************************
    changed: [sonar-01]

    PLAY [Get PostgreSQL installed] ******************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************
    ok: [sonar-01]

    TASK [Change repo file] **************************************************************************************************************************************
    changed: [sonar-01]

    TASK [Install PostgreSQL repos] ******************************************************************************************************************************
    changed: [sonar-01]

    TASK [Install PostgreSQL] ************************************************************************************************************************************
    changed: [sonar-01]

    TASK [Init template1 DB] *************************************************************************************************************************************
    changed: [sonar-01]

    TASK [Start pgsql service] ***********************************************************************************************************************************
    changed: [sonar-01]

    TASK [Create user in system] *********************************************************************************************************************************
    changed: [sonar-01]

    TASK [Create user for Sonar in PostgreSQL] *******************************************************************************************************************
    [WARNING]: Module remote_tmp /var/lib/pgsql/.ansible/tmp did not exist and was created with a mode of 0700, this may cause issues when running as another
    user. To avoid this, create the remote_tmp dir with the correct permissions manually
    changed: [sonar-01]

    TASK [Change password for Sonar user in PostgreSQL] **********************************************************************************************************
    changed: [sonar-01]

    TASK [Create Sonar DB] ***************************************************************************************************************************************
    changed: [sonar-01]

    TASK [Copy pg_hba.conf] **************************************************************************************************************************************
    changed: [sonar-01]

    PLAY [Prepare Sonar host] ************************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************
    ok: [sonar-01]

    TASK [Create group in system] ********************************************************************************************************************************
    ok: [sonar-01]

    TASK [Create user in system] *********************************************************************************************************************************
    ok: [sonar-01]

    TASK [Set up ssh key to access for managed node] *************************************************************************************************************
    changed: [sonar-01]

    TASK [Allow group to have passwordless sudo] *****************************************************************************************************************
    changed: [sonar-01]

    TASK [Increase Virtual Memory] *******************************************************************************************************************************
    changed: [sonar-01]

    TASK [Reboot VM] *********************************************************************************************************************************************
    changed: [sonar-01]

    PLAY [Get Sonarqube installed] *******************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************
    ok: [sonar-01]

    TASK [Get distrib ZIP] ***************************************************************************************************************************************
    changed: [sonar-01]

    TASK [Unzip Sonar] *******************************************************************************************************************************************
    changed: [sonar-01]

    TASK [Move Sonar into place.] ********************************************************************************************************************************
    changed: [sonar-01]

    TASK [Configure SonarQube JDBC settings for PostgreSQL.] *****************************************************************************************************
    changed: [sonar-01] => (item={'regexp': '^sonar.jdbc.username', 'line': 'sonar.jdbc.username=sonar'})
    changed: [sonar-01] => (item={'regexp': '^sonar.jdbc.password', 'line': 'sonar.jdbc.password=sonar'})
    changed: [sonar-01] => (item={'regexp': '^sonar.jdbc.url', 'line': 'sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance'})                                                                                                 
    changed: [sonar-01] => (item={'regexp': '^sonar.web.context', 'line': 'sonar.web.context='})

    TASK [Generate wrapper.conf] *********************************************************************************************************************************
    changed: [sonar-01]

    TASK [Symlink sonar bin.] ************************************************************************************************************************************
    changed: [sonar-01]

    TASK [Copy SonarQube systemd unit file into place (for systemd systems).] ************************************************************************************
    changed: [sonar-01]

    TASK [Ensure Sonar is running and set to start on boot.] *****************************************************************************************************
    changed: [sonar-01]

    TASK [Allow Sonar time to build on first start.] *************************************************************************************************************
    Pausing for 180 seconds
    (ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
    ok: [sonar-01]

    TASK [Make sure Sonar is responding on the configured port.] *************************************************************************************************
    ok: [sonar-01]

    PLAY [Get Nexus installed] ***********************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************
    ok: [nexus-01]

    TASK [Create Nexus group] ************************************************************************************************************************************
    changed: [nexus-01]

    TASK [Create Nexus user] *************************************************************************************************************************************
    changed: [nexus-01]

    TASK [Install JDK] *******************************************************************************************************************************************
    changed: [nexus-01]

    TASK [Create Nexus directories] ******************************************************************************************************************************
    changed: [nexus-01] => (item=/home/nexus/log)
    changed: [nexus-01] => (item=/home/nexus/sonatype-work/nexus3)
    changed: [nexus-01] => (item=/home/nexus/sonatype-work/nexus3/etc)
    changed: [nexus-01] => (item=/home/nexus/pkg)
    changed: [nexus-01] => (item=/home/nexus/tmp)

    TASK [Download Nexus] ****************************************************************************************************************************************
    [WARNING]: Module remote_tmp /home/nexus/.ansible/tmp did not exist and was created with a mode of 0700, this may cause issues when running as another user.
    To avoid this, create the remote_tmp dir with the correct permissions manually
    changed: [nexus-01]

    TASK [Unpack Nexus] ******************************************************************************************************************************************
    changed: [nexus-01]

    TASK [Link to Nexus Directory] *******************************************************************************************************************************
    changed: [nexus-01]

    TASK [Add NEXUS_HOME for Nexus user] *************************************************************************************************************************
    changed: [nexus-01]

    TASK [Add run_as_user to Nexus.rc] ***************************************************************************************************************************
    changed: [nexus-01]

    TASK [Raise nofile limit for Nexus user] *********************************************************************************************************************
    changed: [nexus-01]

    TASK [Create Nexus service for SystemD] **********************************************************************************************************************
    changed: [nexus-01]

    TASK [Ensure Nexus service is enabled for SystemD] ***********************************************************************************************************
    changed: [nexus-01]

    TASK [Create Nexus vmoptions] ********************************************************************************************************************************
    changed: [nexus-01]

    TASK [Create Nexus properties] *******************************************************************************************************************************
    changed: [nexus-01]

    TASK [Lower Nexus disk space threshold] **********************************************************************************************************************
    skipping: [nexus-01]

    TASK [Start Nexus service if enabled] ************************************************************************************************************************
    changed: [nexus-01]

    TASK [Ensure Nexus service is restarted] *********************************************************************************************************************
    skipping: [nexus-01]

    TASK [Wait for Nexus port if started] ************************************************************************************************************************
    ok: [nexus-01]

    PLAY RECAP ***************************************************************************************************************************************************
    nexus-01                   : ok=17   changed=15   unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
    sonar-01                   : ok=35   changed=27   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


Залогинился в Sonarqube и Nexus (скриншоты 16 и 19), сменил пароли.

Создал в Sonarqube проект test1, скачал SonarScanner 4.6.2, добавил в переменные окружения: 

    [max@max_centos bin]$ echo $PATH
    /usr/lib64/qt-3.3/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/home/max/.local/bin:/home/max/bin
    [max@max_centos bin]$ 
    [max@max_centos bin]$ export PATH=$PATH:/home/max/SonarQube/sonar-scanner-4.6.2.2472-linux/bin
    [max@max_centos bin]$ 
    [max@max_centos bin]$ echo $PATH
    /usr/lib64/qt-3.3/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/home/max/.local/bin:/home/max/bin:/home/max/SonarQube/sonar-scanner-4.6.2.2472-linux/bin

    [max@max_centos bin]$ sonar-scanner --version
    INFO: Scanner configuration file: /home/max/SonarQube/sonar-scanner-4.6.2.2472-linux/conf/sonar-scanner.properties
    INFO: Project root configuration file: NONE
    INFO: SonarScanner 4.6.2.2472
    INFO: Java 11.0.11 AdoptOpenJDK (64-bit)
    INFO: Linux 3.10.0-1160.45.1.el7.x86_64 amd64

    Запустил
    sonar-scanner \
      -Dsonar.projectKey=test1 \
      -Dsonar.sources=. \
      -Dsonar.host.url=http://130.193.35.179:9000 \
      -Dsonar.login=6baa446acb13ae9b2052d426c404bac257623d7f
      -Dsonar.coverage.exclusions=fail.py
  
Найдено 2 бага (скриншот 17). Исправил (объявил переменнную и заменил =+ на +=), проверка не нашла багов (скриншот 18).


Скачал jre-8u321-linux-x64 и в Nexus репозиторий maven-public загрузил два артефакта (скриншот 21).


    Скачал Maven 3.8.4, разархивировал, добавил в PATH:
    [max@max_centos bin]$ echo $PATH
    /usr/lib64/qt-3.3/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/home/max/.local/bin:/home/max/bin
    [max@max_centos bin]$ 
    [max@max_centos bin]$ export PATH=$PATH:/home/max/SonarQube/apache-maven-3.8.4/bin
    [max@max_centos bin]$ 
    [max@max_centos bin]$ echo $PATH
    /usr/lib64/qt-3.3/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/home/max/.local/bin:/home/max/bin:/home/max/SonarQube/apache-maven-3.8.4/bin

    Удалил из конфига фрагмент
        <mirror>
          <id>maven-default-http-blocker</id>
          <mirrorOf>external:http:*</mirrorOf>
          <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
          <url>http://0.0.0.0/</url>
          <blocked>true</blocked>
        </mirror>
    
    [max@max_centos bin]$ mvn --version
    Apache Maven 3.8.4 (9b656c72d54e5bacbed989b64718c159fe39b537)
    Maven home: /home/max/SonarQube/apache-maven-3.8.4
    Java version: 1.8.0_312, vendor: Red Hat, Inc., runtime: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-1.el7_9.x86_64/jre
    Default locale: ru_RU, platform encoding: UTF-8
    OS name: "linux", version: "3.10.0-1160.45.1.el7.x86_64", arch: "amd64", family: "unix"

Исправил pom.xml и запустил:

    [max@max_centos mvn]$ pwd
    /home/max/SonarQube/mvn
    [max@max_centos mvn]$ mvn package
    [INFO] Scanning for projects...
    [INFO] 
    [INFO] ---------------------------< netology:java >----------------------------
    [INFO] Building java 8_282
    [INFO] --------------------------------[ jar ]---------------------------------
    Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-resources-plugin/2.6/maven-resources-plugin-2.6.pom
    Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-resources-plugin/2.6/maven-resources-plugin-2.6.pom (8.1 kB at 5.2 kB/s)
    Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-plugins/23/maven-plugins-23.pom
    Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-plugins/23/maven-plugins-23.pom (9.2 kB at 55 kB/s)
    Downloading from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-parent/22/maven-parent-22.pom
    Downloaded from central: https://repo.maven.apache.org/maven2/org/apache/maven/maven-parent/22/maven-parent-22.pom (30 kB at 171 kB/s)
    ..........................
    ..........................
    ..........................
    Downloaded from central: https://repo.maven.apache.org/maven2/commons-lang/commons-lang/2.1/commons-lang-2.1.jar (208 kB at 371 kB/s)
    Downloaded from central: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-utils/3.0/plexus-utils-3.0.jar (226 kB at 395 kB/s)
    [WARNING] JAR will be empty - no content was marked for inclusion!
    [INFO] Building jar: /home/max/SonarQube/mvn/target/java-8_282.jar
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  32.444 s
    [INFO] Finished at: 2022-01-31T22:53:21+04:00
    [INFO] ------------------------------------------------------------------------


    [max@max_centos repository]$ pwd
    /home/max/.m2/repository
    [max@max_centos repository]$ ls -l
    итого 0
    drwxrwxr-x 3 max max 38 янв 31 22:53 backport-util-concurrent
    drwxrwxr-x 3 max max 25 янв 31 22:52 classworlds
    drwxrwxr-x 3 max max 20 янв 31 22:53 com
    drwxrwxr-x 3 max max 25 янв 31 22:52 commons-cli
    drwxrwxr-x 3 max max 26 янв 31 22:53 commons-lang
    drwxrwxr-x 3 max max 33 янв 31 22:53 commons-logging
    drwxrwxr-x 3 max max 19 янв 31 22:52 junit
    drwxrwxr-x 3 max max 19 янв 31 22:53 log4j
    drwxrwxr-x 6 max max 65 янв 31 22:53 org 
