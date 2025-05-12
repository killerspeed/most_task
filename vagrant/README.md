# Changelog (v1):

### Первый вариант конфигурации для запуска и установки сервера на Ubuntu 22.04
Изначальная версия настроена для Ubuntu 20.04, поэтому после установки необходимо выполнить обновление до требуемой версии (22.04).

### *Инструкция по установке 22.04:*
1. Для  подготовки системы перед обновлением до Ubuntu 22.04 добовляем в Vagrantfile следующий shell-провижнер:
``` shell
            node.vm.provision "shell", inline: <<-SHELL
                                sudo apt update && sudo apt upgrade -y
				sudo apt dist-upgrade -y
				sudo apt autoremove -y
				sudo apt install update-manager-core -y
            SHELL 
```
- ``` sudo apt update && sudo apt upgrade -y ```

****apt update*** — обновляет список доступных пакетов и их версий из репозиториев.*

****apt upgrade -y*** — устанавливает все доступные обновления для текущих пакетов (флаг -y автоматически подтверждает).*

- ``` sudo apt dist-upgrade -y ```

*Аналогичен ***upgrade***, но также обрабатывает изменения в зависимостях пакетов (может добавлять или удалять пакеты при необходимости).
Цель: Полное обновление системы, включая ядро и критичные компоненты.*

- ``` sudo apt autoremove -y ```

*Удаляет пакеты, которые больше не нужны.*

- ``` sudo apt install update-manager-core -y ```

*Устанавливает пакет update-manager-core, который содержит утилиту do-release-upgrade*

2. Обновление до Ubuntu 22.04 LTS

- ***Проверьте настройки обновления***

*Убедитесь, что в файле `/etc/update-manager/release-upgrades` стоит `Prompt=lts`*

``` bash
sudo cat /etc/update-manager/release-upgrades
```

- ***Запуск обновления командой:***

``` bash
sudo do-release-upgrade
```
*Важно:*

    Во время процесса следуйте инструкциям на экране.
    Ответьте y (yes) на запросы подтверждения.
    При наличии изменений в конфигурационных файлах выбирайте:
    Y — заменить текущую версию (если не вносили правки вручную)
    N — оставить существующую (если были кастомные настройки)

* ***После завершения***

*Перезагрузите систему:*
``` bash 
sudo reboot
```

*Проверьте версию ОС:*

``` bash 
lsb_release -a 
```

# Changelog (v1.2):
1. **Исправленный и улучшенный вариант конфигурации Vagrant**

- *По итогу тестов было принято решено сразу устанволивтаь 22.04
Меняем строчку:*

`Было: node.vm.box = bento/ubuntu-20.04`

`Стало: node.vm.box = "ubuntu/jammy64"`

# Changelog (v1.2.1):
1. **Установка Jenkins через Vagrant**

_Для установки Jenkins добавьте следующий код в блок `node.vm.provision "shell", inline: <<-SHELL` вашего Vagrantfile:_

``` shell
            node.vm.provision "shell", inline: <<-SHELL
                sudo apt update && sudo apt upgrade -y
				sudo apt dist-upgrade -y
				sudo apt autoremove -y
				sudo apt install -y openjdk-21-jdk
				curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
				sudo apt update
				sudo apt install -y jenkins
				sudo systemctl start jenkins

			SHELL
```
# Changelog (v1.2.2):
1. **Добавлени к конфигурации установку zabbix**

_Блок `node.vm.provision "shell", inline: <<-SHELL` выглядит следующим образом_ 
```shell
node.vm.provision "shell", inline: <<-SHELL
			# Обновление системы
                sudo apt update && sudo apt upgrade -y
				sudo apt dist-upgrade -y
				sudo apt autoremove -y
				sudo apt install -y wget curl gnupg2 software-properties-common
				
			# ========== Установка Jenkins ==========
				sudo apt install -y openjdk-21-jdk
				curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
				sudo apt update
				sudo apt install -y jenkins
				sudo systemctl start jenkins
			# ========== Установка PostgreSQL ==========
				sudo sh -c 'echo "deb https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
				wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
			
			# ========== Установка Zabbix ==========
				wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_6.0+ubuntu22.04_all.deb
				sudo dpkg -i zabbix-release_latest_6.0+ubuntu22.04_all.deb
				sudo apt update
				sudo apt install -y postgresql-15 postgresql-client-15
				sudo -u postgres psql -c "CREATE USER zabbix WITH PASSWORD 'strong_password'"
				sudo -u postgres psql -c "CREATE DATABASE zabbix OWNER zabbix"
				sudo apt install -y zabbix-server-pgsql zabbix-frontend-php php8.1-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
				zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
							
            SHELL
```
2. **Улучшения:**
- Скрипт разделен на секции
- Комментарии

# Changelog (v1.2.3):
1. **Улучшения:**
- Конфигурация Nginx:
  - Редактирование файлa /etc/zabbix/nginx.conf раскомментировав и настройки директивы 'listen' и 'server_name'.
- Мониторинг:
  - Добавлена проверка статусов всех сервисов в конце
  - Перезапуск и включение сервисов
- Конфигурация Zabbix Server
  - Автоматическая конфигурация через sed
  - Указан DBHost и DBPassword
  - Прописаны параметры подключения к БД
2. **Добавление кода**
- _Блок `node.vm.provision "shell", inline: <<-SHELL` выглядит следующим образом_
```shell
node.vm.provision "shell", inline: <<-SHELL
			# Обновление системы
                sudo apt update && sudo apt upgrade -y
				sudo apt dist-upgrade -y
				sudo apt autoremove -y
				sudo apt install -y wget curl gnupg2 software-properties-common
				
			# ========== Установка Jenkins ==========
				sudo apt install -y openjdk-21-jdk
				curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
				sudo apt update
				sudo apt install -y jenkins
				sudo systemctl start jenkins
			# ========== Установка PostgreSQL ==========
				sudo sh -c 'echo "deb https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
				wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
			
			# ========== Установка Zabbix ==========
				wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_6.0+ubuntu22.04_all.deb
				sudo dpkg -i zabbix-release_latest_6.0+ubuntu22.04_all.deb
				sudo apt update
				sudo apt install -y postgresql-15 postgresql-client-15
				sudo apt install -y zabbix-server-pgsql zabbix-frontend-php php8.1-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
				
			# Настройка БД	
				sudo -u postgres psql -c "CREATE USER zabbix WITH PASSWORD 'strong_password'"
				sudo -u postgres psql -c "CREATE DATABASE zabbix OWNER zabbix"
				zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
				
			# Конфигурация Zabbix Server
				sudo sed -i 's/^# DBHost.*/DBHost=localhost/' /etc/zabbix/zabbix_server.conf
				sudo sed -i 's/^# DBPassword.*/DBPassword=strong_password/' /etc/zabbix/zabbix_server.conf
				
			# Конфигурация Nginx	
				sudo sed -i 's/^#        listen          8080;.*/listen 172.18.1.15:80;/' /etc/zabbix/nginx.conf
				sudo sed -i 's/^#        server_name     example.com;.*/server_name 172.18.1.15;/' /etc/zabbix/nginx.conf
				
			# Перезапуск и включение сервисов
				sudo systemctl restart postgresql
				sudo systemctl enable --now zabbix-server zabbix-agent nginx php8.1-fpm
				sudo systemctl restart nginx

			# Проверка статусов
				echo "Проверка статусов сервисов:"
				sudo systemctl status jenkins postgresql zabbix-server zabbix-agent nginx --no-pager
            SHELL
```
# Changelog (v1.3):
1. **Улучшения:**
- Added
  - Поддержка Docker-установки Jenkins
  - Автоматическое развертывание Docker-in-Docker (DinD)
  - Создание сети Docker для изоляции Jenkins
- Changed
  - Переход на нумерацию версий в формате vX.Y.Z
  - Обновление конфигурации Nginx для Zabbix

2. **Добавление кода**
- _Блок `node.vm.provision "shell", inline: <<-SHELL` выглядит следующим образом
```shell
node.vm.provision "shell", inline: <<-SHELL
			# Обновление системы
                sudo apt update && sudo apt upgrade -y
				sudo apt dist-upgrade -y
				sudo apt autoremove -y
				sudo apt install -y wget curl gnupg2 software-properties-common
				
			# ========== Установка Docker ==========
			    sudo apt install apt-transport-https ca-certificates curl software-properties-common
                curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
                sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
                sudo apt update
                sudo apt install docker-ce -y
                sudo systemctl start docker
				docker network create jenkins
				sudo docker run \
						--name jenkins-docker \
						--rm \
						--detach \
						--privileged \
						--network jenkins \
						--network-alias docker \
						--env DOCKER_TLS_CERTDIR=/certs \
						--volume jenkins-docker-certs:/certs/client \
						--volume jenkins-data:/var/jenkins_home \
						--publish 2376:2376 \
						docker:dind \
						--storage-driver overlay2
				cd jenkins/
				docker build -t myjenkins-blueocean:2.504.1-1 .
				docker run \
  --name jenkins-blueocean \
  --restart=on-failure \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.504.1-1
			# ========== Установка PostgreSQL ==========
				sudo sh -c 'echo "deb https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
				wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

			# ========== Установка Zabbix ==========
				wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_6.0+ubuntu22.04_all.deb
				sudo dpkg -i zabbix-release_latest_6.0+ubuntu22.04_all.deb
				sudo apt update
				sudo apt install -y postgresql-15 postgresql-client-15
				sudo apt install -y zabbix-server-pgsql zabbix-frontend-php php8.1-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent

			# Настройка БД
				sudo -u postgres psql -c "CREATE USER zabbix WITH PASSWORD 'strong_password'"
				sudo -u postgres psql -c "CREATE DATABASE zabbix OWNER zabbix"
				zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix

			# Конфигурация Zabbix Server
				sudo sed -i 's/^# DBHost.*/DBHost=localhost/' /etc/zabbix/zabbix_server.conf
				sudo sed -i 's/^# DBPassword.*/DBPassword=strong_password/' /etc/zabbix/zabbix_server.conf

			# Конфигурация Nginx
				sudo sed -i 's/^#        listen          8080;.*/listen 172.18.1.15:80;/' /etc/zabbix/nginx.conf
				sudo sed -i 's/^#        server_name     example.com;.*/server_name 172.18.1.15;/' /etc/zabbix/nginx.conf

			# Перезапуск и включение сервисов
				sudo systemctl restart postgresql
				sudo systemctl enable --now zabbix-server zabbix-agent nginx php8.1-fpm
				sudo systemctl restart nginx

			# Проверка статусов
				echo "Проверка статусов сервисов:"
				sudo systemctl status docker postgresql zabbix-server zabbix-agent nginx --no-pager
            SHELL
```