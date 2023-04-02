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








