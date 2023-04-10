## JENKINS

### Установка Jenkins

Jenkins
https://www.jenkins.io/

Jenkins download and deployment
https://www.jenkins.io/download/

Installing Jenkins
https://www.jenkins.io/doc/book/installing/


Версия ОС:
```
cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

Зададим свой часовой пояс:
```
timedatectl set-timezone Europe/Moscow
```

Установим службу для синхронизации времени:
```
apt install chrony
```

Запустим chrony и разрешим ее автозапуск:
```
systemctl enable --now chrony
```

При настройке firewall (iptables, nftables) необходимо открыть порт 8080, на котором работает Jenkins, в частности:
```
iptables -I INPUT -p tcp —dport 8080 -j ACCEPT
```

Обновим список обновлений:
```
apt update
```

Установим Java:
```
apt install openjdk-11-jre
```

Версия установленной Java:
```
java -version
```

Будет выглядеть примерно так:
```
openjdk version "11.0.18" 2023-01-17
OpenJDK Runtime Environment (build 11.0.18+10-post-Debian-1deb11u1)
OpenJDK 64-Bit Server VM (build 11.0.18+10-post-Debian-1deb11u1, mixed mode, sharing)
```

Собственно, сама установка:
```
wget -qO - https://pkg.jenkins.io/debian/jenkins.io-2023.key | cat - > /usr/share/keyrings/jenkins-keyring.asc
```
```
echo 'deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/' > /etc/apt/sources.list.d/jenkins.list
```
```
apt update
```
```
apt install jenkins
```

Убедимся, что jenkins запустился:
```
systemctl status jenkins.service 
```

В адресной строке браузера вводим http://192.168.56.101:8080:

<img src="./images/Screenshot from 2023-04-02 14-44-47.png" />


Скопируем пароль с файла /var/lib/jenkins/secrets/initialAdminPassword:
```
cat /var/lib/jenkins/secrets/initialAdminPassword
```

и вставим в поле [Administrator password] в браузере и кликаем [Continue]:

<img src="./images/Screenshot from 2023-04-02 16-41-08.png" />


Кликаем [Select plugins to install]:

<img src="./images/Screenshot from 2023-04-02 16-45-11.png" />

<img src="./images/Screenshot from 2023-04-02 16-47-16.png" />


Кликаем "None", то есть убираем все галочки:

<img src="./images/Screenshot from 2023-04-02 17-15-48.png" />


далее [Install]:

<img src="./images/Screenshot from 2023-04-02 17-22-28.png" />


Заполним все поля (для демонстрации пароль: 12345678) и кликаем [Save and Continue]:

<img src="./images/Screenshot from 2023-04-02 17-28-35.png" />


И на следующей странице кликаем [Save and Finish]:

<img src="./images/Screenshot from 2023-04-02 17-29-55.png" />


Далее [Start using Jenkins]:

<img src="./images/Screenshot from 2023-04-02 17-31-45.png" />


Открывается главная страница:

<img src="./images/Screenshot from 2023-04-02 17-33-36.png" />


### Администрирование Jenkins

В "Manage Jenkins" > "System" уберём галочку в "Help make Jenkins better by sending anonymous usage statistics and crash reports to the Jenkins project" и кликаем [Save]:

<img src="./images/Screenshot from 2023-04-02 18-00-27.png" />


В "Manage Jenkins" > "Security" ставим галочку "Disable remember me" и кликаем [Save]:

<img src="./images/Screenshot from 2023-04-02 18-04-49.png" />


Чтобы обновить версию Jenkins, зайдём в директорий, указанный в "Manage Jenkins" > "System Information":

<img src="./images/Screenshot from 2023-04-02 18-29-05.png" />

```
cd /usr/share/java/
```

Переименуем файл jenkins.war, например, в конце имени файла добавим номер версии:
```
mv ./jenkins.war ./jenkins-2.397.war 
```

и в этот директорий скачаем новую версию, например:
```
wget -O jenkins.war http://update.jenkins-ci.org/download/war/2.398/jenkins.war
```

Перезапускаем Jenkins сервис:
```
systemctl restart jenkins 
```

### Управление Plugins

Для установки плагина, например, Git, зайдём на страницу https://www.jenkins.io/  и кликаем по ссылке "Plugins", затем в поле набираем "git':

<img src="./images/Screenshot from 2023-04-02 21-59-26.png" />


Затем выбираем самый первый, где большее число установок:

<img src="./images/Screenshot from 2023-04-02 22-08-34.png" />


Кликаем по ссылке "Releases":

<img src="./images/Screenshot from 2023-04-02 22-12-20.png" />


Кликаем правой кнопкой мыши по ссылке "direct link":

<img src="./images/Screenshot from 2023-04-02 22-27-35.png" />


В контекстном меню выбираем [Copy link].

На странице управления Jenkins в "Manage Jenkins" > "Plugins" > "Advanced settings", в секции "Deploy Plugin" в поле URL вставляем скоипрованную ссылку и кликаем [Deploy]:

<img src="./images/Screenshot from 2023-04-02 22-37-15.png" />


Идёт процесс установки выбранного плагина и его зависимости:

<img src="./images/Screenshot from 2023-04-02 22-39-28.png" />


Ставим галочку "Restart Jenkins when installation is complete and no jobs are running":

<img src="./images/Screenshot from 2023-04-02 22-40-56.png" />


После рестарта авторизуемся:

<img src="./images/Screenshot from 2023-04-02 22-47-19.png" />


Заходим в "Manage Jenkins" > "Plugins" > "Installed plugins", убедимся, что плагин Git установлен:

<img src="./images/Screenshot from 2023-04-02 22-52-34.png" />


### Простейшие Jobs, включая Deployment

Создадим новый job. Кликаем на [Create a job]:

<img src="./images/Screenshot from 2023-04-03 19-35-15.png" />

или [+ New Item]:

<img src="./images/Screenshot from 2023-04-03 19-36-16.png" />


Вводим имя нового job, например, MyJob-01, выбираем "Freestyle project" и нажимаем [OK]:

<img src="./images/Screenshot from 2023-04-03 19-43-42.png" />


Вводим пока Description, например:
```
This is my first ever Jenkins job!
```

<img src="./images/Screenshot from 2023-04-03 19-48-47.png" />

<img src="./images/Screenshot from 2023-04-03 19-49-17.png" />


Из списка [Build step] выбираем "Execute shell":

<img src="./images/Screenshot from 2023-04-03 19-54-43.png" />


В поле "Command" вводим какую-нибудь стандартную команду Linux, например:
```
echo "Hello World!"
```
<img src="./images/Screenshot from 2023-04-03 19-58-47.png" />

и нажимаем [Save].

<img src="./images/Screenshot from 2023-04-03 20-01-13.png" />

Теперь на главной странице отображается новый job:

<img src="./images/Screenshot from 2023-04-03 20-02-32.png" />


Для запуска нужно кликнуть [Build Now]:

<img src="./images/Screenshot from 2023-04-03 20-31-11.png" />


Для того чтобы увидеть результат действия job, кликаем [Console Output]:

<img src="./images/Screenshot from 2023-04-03 20-34-34.png" />

И видим в консоли вывод "Hello World!":

<img src="./images/Screenshot from 2023-04-03 20-37-13.png" />


Добавим в этот job что-нибудь ещё:

<img src="./images/Screenshot from 2023-04-03 20-43-12.png" />

Если кликнуть по ссылке "See the list of available environment variables":

<img src="./images/Screenshot from 2023-04-03 20-45-33.png" />

то увидим список переменных:

<img src="./images/Screenshot from 2023-04-03 20-47-22.png" />


Вот и добавим в "Build Steps" - "Execute shell" - "Command" ещё строку, например:
```
echo "This is Build number $BUILD_NUMBER"
```
<img src="./images/Screenshot from 2023-04-03 20-55-18.png" />

и кликаем [Save].


Запустим Build [Build Now] и смотрим результат вывода [Console Output]:

<img src="./images/Screenshot from 2023-04-03 21-00-23.png" />


Добавим ещё несколько строк в "Build Steps" - "Execute shell" - "Command":
```
echo "Hello World!"
echo "This is Build number $BUILD_NUMBER"
pwd
sleep 5
whoami
sleep 5
echo "Nmae of this Build is $BUILD_DISPLAY_NAME"
```
<img src="./images/Screenshot from 2023-04-03 21-13-53.png" />

Для того чтобы build-ы моглы выполняться одновременно, в настройке Job поставим галочку в "Execute concurrent builds if necessary":

<img src="./images/Screenshot from 2023-04-03 21-18-33.png" />

и нажимаем [Save].

Несколько раз нажмём "Build Now" и смотрим "Build Executor Status" в левой части страницы:

<img src="./images/Screenshot from 2023-04-03 21-23-18.png" />

Наблюдаем параллельное выполнение build-ов.

Результат вывода:

<img src="./images/Screenshot from 2023-04-03 21-26-52.png" />


Добавим команду, которая выведет ошибку, например:
```
cat /etc/nofoundfile.txt
```
т.е. попытаемся вывести на экран содержимое несуществующего файла.

<img src="./images/Screenshot from 2023-04-03 21-32-54.png" />


Запустим [Build Now] и наблюдаем, что вместо зелёной галочки в кружочке получим красный крестик, и появилось уведомление в графе "Last Failure":

<img src="./images/Screenshot from 2023-04-03 21-38-02.png" />

Смотрим консоль [Console Output]:

<img src="./images/Screenshot from 2023-04-03 21-40-38.png" />

10:22

Теперь удалим строку с командой, вызывающую ошибку.


Установим плагин, который добавит в правой и нижней части страницы изображение. Например, плагин "ChuckNorris" с изображением Чака Норриса. Затем в конфигурации Job "Configure" - "Post-build Actions" из списка выбираем "Activate ChuckNorris" 

<img src="./images/Screenshot from 2023-04-04 20-46-19.png" />

и нажимаем [Save] и получим следующее:

<img src="./images/Screenshot from 2023-04-04 20-49-05.png" />


Файлы Jenkis хранятся в директории /var/lib/jenkins/:
```
root@server1:~# ls -l /var/lib/jenkins/
total 124
-rw-r--r--  1 jenkins jenkins 1800 Apr  4 20:30 config.xml
-rw-r--r--  1 jenkins jenkins  156 Apr  4 20:30 hudson.model.UpdateCenter.xml
-rw-r--r--  1 jenkins jenkins  370 Apr  2 22:39 hudson.plugins.git.GitTool.xml
-rw-r--r--  1 jenkins jenkins   76 Apr  2 18:02 hudson.tasks.Shell.xml
-rw-r--r--  1 jenkins jenkins  216 Apr  2 18:02 hudson.triggers.SCMTrigger.xml
-rw-------  1 jenkins jenkins 1680 Apr  2 22:39 identity.key.enc
-rw-r--r--  1 jenkins jenkins  277 Apr  2 18:02 jenkins.fingerprints.GlobalFingerprintConfiguration.xml
-rw-r--r--  1 jenkins jenkins    5 Apr  4 20:30 jenkins.install.InstallUtil.lastExecVersion
-rw-r--r--  1 jenkins jenkins    5 Apr  2 17:33 jenkins.install.UpgradeWizard.state
-rw-r--r--  1 jenkins jenkins  159 Apr  2 18:02 jenkins.model.ArtifactManagerConfiguration.xml
-rw-r--r--  1 jenkins jenkins  253 Apr  2 18:02 jenkins.model.GlobalBuildDiscarderConfiguration.xml
-rw-r--r--  1 jenkins jenkins  265 Apr  2 18:02 jenkins.model.JenkinsLocationConfiguration.xml
-rw-r--r--  1 jenkins jenkins  169 Apr  2 18:09 jenkins.security.QueueItemAuthenticatorConfiguration.xml
-rw-r--r--  1 jenkins jenkins   86 Apr  2 18:02 jenkins.security.ResourceDomainConfiguration.xml
-rw-r--r--  1 jenkins jenkins  162 Apr  2 18:09 jenkins.security.UpdateSiteWarningsConfiguration.xml
-rw-r--r--  1 jenkins jenkins  357 Apr  2 18:09 jenkins.security.apitoken.ApiTokenPropertyConfiguration.xml
-rw-r--r--  1 jenkins jenkins  179 Apr  2 18:02 jenkins.tasks.filters.EnvVarsFilterGlobalConfiguration.xml
-rw-r--r--  1 jenkins jenkins  171 Apr  2 13:27 jenkins.telemetry.Correlator.xml
drwxr-xr-x  3 jenkins jenkins 4096 Apr  3 19:46 jobs
drwxr-xr-x  3 jenkins jenkins 4096 Apr  2 15:20 logs
-rw-r--r--  1 jenkins jenkins  907 Apr  4 20:30 nodeMonitors.xml
drwxr-xr-x  2 jenkins jenkins 4096 Apr  2 13:27 nodes
drwxr-xr-x 28 jenkins jenkins 4096 Apr  4 20:40 plugins
-rw-r--r--  1 jenkins jenkins  129 Apr  4 20:40 queue.xml
-rw-r--r--  1 jenkins jenkins  129 Apr  3 21:45 queue.xml.bak
-rw-r--r--  1 jenkins jenkins   64 Apr  2 13:27 secret.key
-rw-r--r--  1 jenkins jenkins    0 Apr  2 13:27 secret.key.not-so-secret
drwx------  2 jenkins jenkins 4096 Apr  3 20:36 secrets
drwxr-xr-x  2 jenkins jenkins 4096 Apr  4 20:31 updates
drwxr-xr-x  2 jenkins jenkins 4096 Apr  2 13:27 userContent
drwxr-xr-x  3 jenkins jenkins 4096 Apr  2 17:29 users
drwxr-xr-x  4 jenkins jenkins 4096 Apr  3 21:23 workspace
root@server1:~#
```

Особенно растёт количество файлов при запуска job в директории /var/lib/jenkins/jobs:
```
root@server1:~# ls -l /var/lib/jenkins/jobs/
total 4
drwxr-xr-x 3 jenkins jenkins 4096 Apr  4 20:46 MyJob-01
root@server1:~#
```

```
root@server1:~# ls -l /var/lib/jenkins/jobs/MyJob-01/builds/
total 32
drwxr-xr-x 2 jenkins jenkins 4096 Apr  3 20:32 1
drwxr-xr-x 2 jenkins jenkins 4096 Apr  3 20:59 2
drwxr-xr-x 2 jenkins jenkins 4096 Apr  3 21:23 3
drwxr-xr-x 2 jenkins jenkins 4096 Apr  3 21:23 4
drwxr-xr-x 2 jenkins jenkins 4096 Apr  3 21:23 5
drwxr-xr-x 2 jenkins jenkins 4096 Apr  3 21:34 6
drwxr-xr-x 2 jenkins jenkins 4096 Apr  4 20:39 7
-rw-r--r-- 1 jenkins jenkins    0 Apr  3 19:46 legacyIds
-rw-r--r-- 1 jenkins jenkins  124 Apr  4 20:39 permalinks
root@server1:~# 
```

где мы видим директории с числовыми именами, то есть с нумерацией build-ов.

Чтобы они так много не копились, в конфигурации поставим галочку в "Discard old builds", в "Strategy" - "Log Rotation", в "Max # of builds to keep" поставим, например, 5, то есть будем хранить пять последних build-ов, и нажмём [Save]:

<img src="./images/Screenshot from 2023-04-04 21-16-53.png" />


Создадим новый job "Deploy-to-TEST":

<img src="./images/Screenshot from 2023-04-04 21-25-39.png" />

<img src="./images/Screenshot from 2023-04-04 21-29-53.png" />

<img src="./images/Screenshot from 2023-04-04 21-31-27.png" />

В "Build Steps" - "Execute shell" добавим два скрипта:

```
echo "-----------Build Started-------------"
cat <<EOF > index.html
<html>
<body bgcolor=black>
<center>
<h2><font color=yellow>Hello from SergSha</font></h2>
<font color=blue>www.sergsha.net</font>
</center>
</body>
</html>
EOF
echo "-----------Build Finished------------"
```

<img src="./images/Screenshot from 2023-04-04 22-09-22.png" />

```
echo "-----------Test Started-------------"
result=$(grep "Hello" index.html | wc -l)
echo $result
if [ "$result" = "1" ]
then
    echo "Test Passed"
    #exit 0
else
    echo "Test Failed"
    exit 1
fi
echo "-----------Test Finished------------"
```

<img src="./images/Screenshot from 2023-04-04 22-10-09.png" />

и нажимаем [Save].


Запустим этот job "Deploy-to-TEST" и смотрим результат выполнения:

<img src="./images/Screenshot from 2023-04-04 22-11-39.png" />

Как видим, тест прошёл успешно.

23:44


Теперь будем делать Deployment.

Установим плагин "Publish Over SSH".


Для начала нужно убедиться, что на серверах server2 и server3 установлены и запущены web сервисы, например, apache2 или nginx. 
Для установки apache2 на debian:
```
apt update
apt install apache2 -y
```

Файл web-страницы расположен по пути:
```
/var/www/html/index.html
```

Введя ip адрес сервера в строке браузера после установки apache2 web-страница должна выглядеть следующим образом:

<img src="./images/Screenshot from 2023-04-08 08-39-45.png" />


Для сервиса Jenkis сгенерируем пару ssh-ключей для подключения к серверам server2 и server3:
```
mkdir /var/lib/jenkins/.ssh
ssh-keygen -t rsa -C "jenkins" -m PEM -P "" -f /var/lib/jenkins/.ssh/id_rsa
chown -R jenkins: /var/lib/jenkins/.ssh
```
Содержимое открытого ключа id_rsa.pub разместим в authorized_keys в серверах server2 и server3.

Чтобы не запрашивало разрешение на подключение к серверам server2 и server3, на Jenkins сервере server1 в файл known_hosts добавим отпечатки от серверов server2 и server3:
```
ssh-keyscan -t rsa 192.168.50.102 >> ./.ssh/known_hosts
ssh-keyscan -t rsa 192.168.50.103 >> ./.ssh/known_hosts
```


Отконфирурируем эти сервера в Jenkins: "Manage Jenkins" > "System", спускаемся до секции "Publish over SSH" и в поле "Path to key" вставим путь до приватного ключа для доступа к серверам server2 и server3:
```
/var/lib/jenkins/.ssh/id_rsa
```
и в "SSH Servers" кликаем [Add]:

<img src="./images/Screenshot from 2023-04-08 14-38-05.png" />

Заполняем поля для server2:

Name:
```
WebServer-TEST
```

Hostname:
```
192.168.50.102
```

Username:
```
root
```

Remote Directory:
```
/var/www/html/
```

кликаем по "Test Configuration" для проверки подключения к серверу server2, результатом которого должен быть "Success":

<img src="./images/Screenshot from 2023-04-08 14-46-08.png" />


Кликаем [Add], чтобы добавить данные для подключения к server3:

Name:
```
WebServer-PROD
```

Hostname:
```
192.168.50.103
```

Username:
```
root
```

Remote Directory:
```
/var/www/html/
```

также кликаем по "Test Configuration" для проверки подключения к серверу server3, результатом которого должен быть "Success":

<img src="./images/Screenshot from 2023-04-08 14-51-00.png" />


Кликаем [Save]


Заходим в конфигурацию job "Deploy-to-Test": "Deploy-to-TEST' > "Configure" и в самом внизу, в секции "Post-build Actions" (если этот шаг будет последним), кликаем [Add post-build action]:

<img src="./images/Screenshot from 2023-04-08 08-55-19.png" />

Из списка выбираем "Send build artifacts over SSH":

<img src="./images/Screenshot from 2023-04-08 08-58-28.png" />


В случае, если этот шаг будет НЕ последним, то можно воспользоваться тем же [Add build step] и из списка выбираем "Send files or execute commands over SSH":

<img src="./images/Screenshot from 2023-04-08 09-06-35.png" />


Заполняем необходимые поля:

Name
```
WebServer-TEST
```

Source files
```
*
```

<img src="./images/Screenshot from 2023-04-08 14-58-37.png" />

Exec command
```
sudo systemctl reload apache2
```

и кликаем [Save]:

<img src="./images/Screenshot from 2023-04-08 15-03-19.png" />


Теперь запускаем job "Deploy-to-TEST".

В адресной строке браузера вводим:
```
192.168.50.102
```

<img src="./images/Screenshot from 2023-04-08 15-06-48.png" />

Результат выполнения в консоли:

<img src="./images/Screenshot from 2023-04-08 16-02-37.png" />


Изменим в конфигурации job "Deploy-to-TEST" фон страницы с черного на белый:

<img src="./images/Screenshot from 2023-04-08 16-08-40.png" />

кликаем [Save] и снова запускаем job "Deploy-to-TEST".

Обновим страницу в браузере, где должны увидеть смену фона с чёрного на белый:

<img src="./images/Screenshot from 2023-04-08 16-13-48.png" />

Результат выполнения в консоли:

<img src="./images/Screenshot from 2023-04-08 16-15-55.png" />

Как видим, deployment TEST прошёл успешно, теперь сделаем deployment PRODUCTION.

Создадим новый job "Deploy-to-PROD".

Для создания нового job на главной странице кликаем [+ New Item]:

<img src="./images/Screenshot from 2023-04-08 16-21-38.png" />

В поле "Enter an item name" вводим новое название:
```
Deploy-to-PROD
```

В поле "Copy from" выбираем существующий job "Deploy-to-TEST", то есть тем самым создаём новый job "Deploy-to-PROD" на базе job "Deploy-to-TEST", и кликаем [OK]:

<img src="./images/Screenshot from 2023-04-08 16-29-07.png" />

Затем в конфигурации job "Deploy-to-PROD", в секции "Post-build Actions", в поле "Name" в подсекции "SSH Server" изменим с "Deploy-to-TEST" на "Deploy-to-PROD" и нажимаем [Save]:

<img src="./images/Screenshot from 2023-04-08 16-33-43.png" />

Перед тем как запустить новый job "Deploy-to-PROD", для демонстрации изменим цвет шрифта с голубого (blue) на пурпурный (magenta) и нажимаем [Save]:

<img src="./images/Screenshot from 2023-04-08 16-47-03.png" />

Далее на главной странице запустим новый job "Deploy-to-PROD":

<img src="./images/Screenshot from 2023-04-08 16-49-36.png" />

В консоли смотрим результат выполнения:

<img src="./images/Screenshot from 2023-04-08 16-52-23.png" />

В адресной строке браузера вводим ip адрес сервера server3:
```
192.168.50.103
```

<img src="./images/Screenshot from 2023-04-08 16-54-23.png" />

Как видим, всё прошло успешно.

Чтобы случайно не запустить job "Deploy-to-PROD", в конфигурции, в самом верху, переводим переключатель с "Enabled" на "Disabled" и нажимаем [Save]:

<img src="./images/Screenshot from 2023-04-08 17-12-14.png" />

Данный job "Deploy-to-PROD" станет неактивным для запуска:

<img src="./images/Screenshot from 2023-04-08 17-16-56.png" />


### Добавление Slave | Node | Agent

Предварительно подготовим ещё два сервера node1 и node2. 
В них должны быть установлены java:
```
apt update
apt install openjdk-11-jre -y
```

В этих серверах в authorized_keys должны быть добавлены публичные ключи. 
Отпечатки этих серверов должны быть добавлены в known_hosts ведущего сервера server1. 
Эти отпечатки можно добавить следующим образом:
```
ssh-keyscan -t rsa 192.168.50.111 >> ./.ssh/known_hosts
ssh-keyscan -t rsa 192.168.50.112 >> ./.ssh/known_hosts
```

Для добавления нод дополнительно установим плагины "SSH Agent":

<img src="./images/Screenshot from 2023-04-08 17-41-40.png" />

и "SSH Build Agents":

<img src="./images/Screenshot from 2023-04-08 18-43-53.png" />

Заходим в "Manage Jenkins" > "Nodes and Clouds" и кликаем [+ New Node]:

<img src="./images/Screenshot from 2023-04-08 18-15-09.png" />

В поле "Node name":
```
Node1
```
выбираем "Permanent Agent" и нажимаем [Create]:

<img src="./images/Screenshot from 2023-04-08 18-17-56.png" />

Вводим необходимые поля:

Description
```
This is Debian Node-1
```

Number of executors
```
2
```

Remote root directory
```
/root/jenkins/
```

Labels (Очень важная и нужная вещь! Вводим несколько через пробел)
```
debian debian_ansible debian_node1
```

<img src="./images/Screenshot from 2023-04-08 18-25-31.png" />

Usage (пока оставляем эту опцию)
```
Use this node as much as possible
```

Launch method
```
Launch agent via SSH
```

Host
```
192.168.50.111
```

<img src="./images/Screenshot from 2023-04-08 18-59-34.png" />

В "Credential" кликаем [Add] и выбираем "Jenkins":

<img src="./images/Screenshot from 2023-04-08 19-01-00.png" />

Domain
```
Global credentials (unrestricted)
```

Kind
```
SSH Username with private key
```

Scope
```
Global (Jenkins, nodes, items, all child items, etc)
```

ID (настоятельно рекомендуется указывать)
```
ssh-key-sss
```

Description
```
ssh-key-sss
```

<img src="./images/Screenshot from 2023-04-08 19-11-41.png" />

Username
```
root
```

В "Private Key" выбираем "Enter directly" и в поле "Key" помещаем содержимое приватного ключа для подключения к нодам node1 и node2, кликаем [Add]:

<img src="./images/Screenshot from 2023-04-08 19-17-09.png" />

В "Credentials" из списка выбираем "root (ssh-key-sss)":

<img src="./images/Screenshot from 2023-04-08 19-21-06.png" />

Host Key Verification Strategy
```
Manually trusted key Verification Strategy
```

Availability (оставим как есть)
```
Keep this agent online as much as possible
```

Кликаем [Save]:

<img src="./images/Screenshot from 2023-04-08 19-28-31.png" />

Сервер node1 добавлен в Jenkins:

<img src="./images/Screenshot from 2023-04-08 20-39-36.png" />


Аналогичным образом подключаем сервер node2.

В "Manage Jenkins" > "Nodes and Clouds" и кликаем [+ New Node], в поле "Node name":
```
Node2
```
выбираем "Permanent Agent" и нажимаем [Create]:

<img src="./images/Screenshot from 2023-04-08 20-58-15.png" />

Вводим необходимые поля:

Description
```
This is Debian Node-2
```

Number of executors
```
2
```

Remote root directory
```
/root/jenkins/
```

Labels (Очень важная и нужная вещь! Вводим несколько через пробел)
```
debian debian_chef debian_node2
```

<img src="./images/Screenshot from 2023-04-08 21-00-10.png" />

Usage (пока оставляем эту опцию)
```
Use this node as much as possible
```

Launch method
```
Launch agent via SSH
```

Host
```
192.168.50.112
```

В "Credentials" из списка выбираем "root (ssh-key-sss)":

<img src="./images/Screenshot from 2023-04-08 21-01-00.png" />

Host Key Verification Strategy
```
Manually trusted key Verification Strategy
```

Availability (оставим как есть)
```
Keep this agent online as much as possible
```

Кликаем [Save]:

<img src="./images/Screenshot from 2023-04-08 21-01-31.png" />

Сервер node2 также успешно добавлен в Jenkins:

<img src="./images/Screenshot from 2023-04-08 21-12-10.png" />


Запустим job "MyJob-01':

<img src="./images/Screenshot from 2023-04-08 21-36-53.png" />

Как видим, build шёл на мастере Jenkins.

Теперь запустим job "MyJob-01' несколько раз:

<img src="./images/Screenshot from 2023-04-08 22-02-48.png" />

Как видим, build-ы шли и на мастере Jenkins, и на нодах node1, node2.

Настроим, чтобы build-ы шли только на нодах node1 и node2. 
Для этого в конфигурации job "MyJob-01" поставим галочку в "Restrict where this project can be run", в поле "Label" введём "debian", метка которой есть и на ноде node1, и на ноде node2:

<img src="./images/Screenshot from 2023-04-08 22-08-13.png" />

и кликаем [Save].

Запустим job "MyJob-01' несколько раз:

<img src="./images/Screenshot from 2023-04-08 22-14-47.png" />

Теперь, как видим, build-ы идут только на нодах node1, node2.

Настроим конфигурацию job "MyJob-01", чтобы build-ы шли только на ноде node1. Для этого в поле "Label" оставим только "debian_node1":

<img src="./images/Screenshot from 2023-04-08 22-19-38.png" />

Снова запустим job "MyJob-01' несколько раз:

<img src="./images/Screenshot from 2023-04-08 22-21-30.png" />

Видим, что теперь build-ы идут только на ноде node1.


### Удалённое и локальное управление через Jenkins CLI Client

Заходим в "Manage Jenkins" > "Jenkins CLI" и видим строку Jenkins CLI и ниже список команд:
```
java -jar jenkins-cli.jar -s http://192.168.50.101:8080/ help
```

<img src="./images/Screenshot from 2023-04-10 19-25-47.png" />

Нажав по ссылке "jenkins-cli.jar" скачается файл jenkins-cli.jar на наш компьютер:
```
http://192.168.50.101:8080/jnlpJars/jenkins-cli.jar
```

Скачаем этот файл локально на Jenkins сервере server1:
```
wget localhost:8080/jnlpJars/jenkins-cli.jar
```

Запустим команду:
```
ls -lh
```
получаем вывод:
```
total 3.3M
-rw-r--r-- 1 root root 3.3M Apr  7 22:12 jenkins-cli.jar
```

Для запуска jenkins cli команды локально используется следующий образец, например, для команды who-am-i:
```
java -jar jenkins-cli.jar -auth username:password -s http://localhost:8080 who-am-i
```

Создадим нового пользователя, например, serviceuser паролем password123 для демонстрации.

Для этого зайдем в "Manage Jenkins" > "Users"

Кликаем [+ Create User]:

<img src="./images/Screenshot from 2023-04-10 19-49-31.png" />

Заполняем необходимые поля:

Username
```
serviceuser
```

Password
```
password123
```

Confirm password
```
password123
```

Full name
```
Service User
```

E-mail address
```
service@something.net
```

и жмём [Create User]

<img src="./images/Screenshot from 2023-04-10 20-03-39.png" />

<img src="./images/Screenshot from 2023-04-10 20-23-16.png" />

Попробуем запустить jenkins cli команду:
```
java -jar jenkins-cli.jar -auth serviceuser:password123 -s http://localhost:8080 who-am-i
```
получаем вывод:
```
Authenticated as: serviceuser
Authorities:
  authenticated
```

Вместо пароля обычно используется API токен.

Чтобы создать токен для пользователя serviceuser, нужно залогиниться под его именем

<img src="./images/Screenshot from 2023-04-10 20-24-36.png" />

Заходим в "Manage Jenkins" > "Users", в конфигурацию пользователя serviceuser

<img src="./images/Screenshot from 2023-04-10 20-28-59.png" />

в секции "API Token" кликаем [Add new Token]

<img src="./images/Screenshot from 2023-04-10 20-32-26.png" />

В поле вводим, например
```
token-01
```
и жмём [Generate]

<img src="./images/Screenshot from 2023-04-10 20-36-04.png" />

скопируем полученный токен
```
11d3cdc14c8502215755b3aa8b1fb6ba6e
```

<img src="./images/Screenshot from 2023-04-10 20-37-13.png" />

Теперь повторим в консоли предыдущую команду, но вместо пароля используем токен:
```
java -jar jenkins-cli.jar -auth serviceuser:11d3cdc14c8502215755b3aa8b1fb6ba6e -s http://localhost:8080 who-am-i
```
как видим, вывел тот же результат:
```
Authenticated as: serviceuser
Authorities:
  authenticated
```

Чтобы каждый раз не вписывать пользователь и токен, воспользуем Environment variables: 
Установка Credential в Linux:
```
export JENKINS_USER_ID=serviceuser
export JENKINS_API_TOKEN=11d3cdc14c8502215755b3aa8b1fb6ba6e
```
Убедимся, что они экспортировались:
```
env | grep JENKINS
```
на экране отобразилось следующее:
```
JENKINS_API_TOKEN=11d3cdc14c8502215755b3aa8b1fb6ba6e
JENKINS_USER_ID=serviceuser
```
Теперь в консоли можем повторно запустить предыдущую команду, но без параметра -auth:
```
java -jar jenkins-cli.jar -s http://localhost:8080 who-am-i
```

Теперь запустим jenkins cli команду НЕ локально, а удалённо.

Скачаем jenkins-cli.jar:
```
wget http://192.168.50.101:8080/jnlpJars/jenkins-cli.jar
```
Также экспортируем необходимые переменные:
```
export JENKINS_USER_ID=serviceuser
export JENKINS_API_TOKEN=11d3cdc14c8502215755b3aa8b1fb6ba6e
```

В Powershell (в ОС Windows) переменные экспортируются следующим образом:
Установка Credential в Windows Powershell:
```
$env:JENKINS_USER_ID="serviceuser"
$env:JENKINS_API_TOKEN="11d3cdc14c8502215755b3aa8b1fb6ba6e"
```

Запустим в консоли команду:
```
java -jar jenkins-cli.jar -s http://192.168.50.101:8080 who-am-i
```
на экран вывел тот же результат:
```
Authenticated as: serviceuser
Authorities:
  authenticated
```

Сохраним job "MyJob-01" в xml-файл:
```
java -jar jenkins-cli.jar -s http://192.168.50.101:8080 get-job MyJob-01 > MyJob-01.xml
```
Содержимое файла MyJob-01:
```
<?xml version='1.1' encoding='UTF-8'?>
<project>
  <actions/>
  <description>This is my first ever Jenkins job!</description>
  <keepDependencies>false</keepDependencies>
  <properties/>
  <scm class="hudson.scm.NullSCM"/>
  <assignedNode>debian_node1</assignedNode>
  <canRoam>false</canRoam>
  <disabled>false</disabled>
  <blockBuildWhenDownstreamBuilding>false</blockBuildWhenDownstreamBuilding>
  <blockBuildWhenUpstreamBuilding>false</blockBuildWhenUpstreamBuilding>
  <triggers/>
  <concurrentBuild>true</concurrentBuild>
  <builders>
    <hudson.tasks.Shell>
      <command>echo &quot;Hello World!&quot;
echo &quot;This is Build number $BUILD_NUMBER&quot;
pwd
sleep 7
whoami
sleep 7
echo &quot;Nmae of this Build is $BUILD_DISPLAY_NAME&quot;
ip address
hostname
sleep 7</command>
      <configuredLocalRules/>
    </hudson.tasks.Shell>
  </builders>
  <publishers/>
  <buildWrappers/>
</project>
```

Для демонстрации внесём изменение в этом файле:
```
<?xml version='1.1' encoding='UTF-8'?>
<project>
  <actions/>
  <description>This is my first ever Jenkins job!</description>
  <keepDependencies>false</keepDependencies>
  <properties/>
  <scm class="hudson.scm.NullSCM"/>
  <assignedNode>debian_node1</assignedNode>
  <canRoam>false</canRoam>
  <disabled>false</disabled>
  <blockBuildWhenDownstreamBuilding>false</blockBuildWhenDownstreamBuilding>
  <blockBuildWhenUpstreamBuilding>false</blockBuildWhenUpstreamBuilding>
  <triggers/>
  <concurrentBuild>true</concurrentBuild>
  <builders>
    <hudson.tasks.Shell>
      <command>echo &quot;Hello World! This string added from XML file and imported via CLI&quot;
echo &quot;This is Build number $BUILD_NUMBER&quot;
pwd
sleep 7
whoami
sleep 7
echo &quot;Nmae of this Build is $BUILD_DISPLAY_NAME&quot;
ip address
hostname
sleep 7</command>
      <configuredLocalRules/>
    </hudson.tasks.Shell>
  </builders>
  <publishers/>
  <buildWrappers/>
</project>
```

Создадим новый job "MyJobFromCLI", импортировав с изменённого xml-файла MyJob-01:
```
java -jar jenkins-cli.jar -s http://192.168.50.101:8080 create-job MyJobFromCLI < MyJob-01.xml
```

В случае в Poweshell (ОС Windows):
```
Get-Content MyJob-01.xml | java -jar jenkins-cli.jar -s http://192.168.50.101:8080 create-job MyJobFromCLI
```

На главной странице увидим только что созданный job "MyJobFromCLI"

<img src="./images/Screenshot from 2023-04-10 21-36-24.png" />

Запустим этот новый job "MyJobFromCLI" и посмотрим результат выполнения:

<img src="./images/Screenshot from 2023-04-10 21-40-51.png" />







