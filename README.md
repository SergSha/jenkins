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

Переименуем файл jenkins.war, например, в конце имени файла добавим ноиер версии:
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


### Просейшие Jobs, включая Deployment

Создадим новый job. Кликаем на [Create a job]:

<img src="./images/Screenshot from 2023-04-03 19-35-15.png" />

или [+ New Item]:

<img src="./images/Screenshot from 2023-04-03 19-36-16.png" />


Вводим имя нового job, например, MyJob-01, выбираем "Freestyle project" и нажимаем [OK]:

<img src="./images/Screenshot from 2023-04-03 19-43-42.png" />


Вводим пока Description:

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















