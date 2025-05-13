# Установка и настройка Zabbix
## нативаня устанвока 
1. Подготовка сервера
Заходим на сервер и выполнянем по очереди команды:

### Скачиваем пакет репозитория
``` bash
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_6.0+ubuntu22.04_all.deb
```

### Устанавливаем репозиторий
``` bash
sudo dpkg -i zabbix-release_latest_6.0+ubuntu22.04_all.deb
```

### Обновляем список пакетов
``` bash
sudo apt update
```

2. Установка PostgreSQL 15

### Добавляем официальный репозиторий PostgreSQL
``` bash
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```

### Импортируем ключ репозитория
``` bash
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

### Устанавливаем PostgreSQL
``` bash
sudo apt install -y postgresql-15 postgresql-client-15
```
3. Установка компонентов Zabbix

### Устанавливаем основные компоненты
``` bash
sudo apt install -y zabbix-server-pgsql zabbix-frontend-php php8.1-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
```

4. Настройка базы данных

### Создаем пользователя и БД
``` bash
sudo -u postgres psql -c "CREATE USER zabbix WITH PASSWORD 'strong_password'"
```
``` bash
sudo -u postgres psql -c "CREATE DATABASE zabbix OWNER zabbix"
```
  
### Импортируем схему

``` bash
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```

5. Конфигурация Zabbix Server

### Настраиваем подключение к БД

``` bash
sudo sed -i 's/^# DBHost.*/DBHost=localhost/' /etc/zabbix/zabbix_server.conf
```
``` bash
sudo sed -i 's/^# DBPassword.*/DBPassword=strong_password/' /etc/zabbix/zabbix_server.conf
```

6. Настройка веб-интерфейса

### Настраиваем nginx

``` bash
sudo sed -i 's/^#        listen          8080;.*/listen 172.18.1.15:80;/' /etc/zabbix/nginx.conf
```
``` bash
sudo sed -i 's/^# DBPassword.*/DBPassword=strong_password/' /etc/zabbix/zabbix_server.conf
```

7. Запуск сервисов

### Перезапускаем PostgreSQL
``` bash
sudo systemctl restart postgresql
```


### Включаем и запускаем сервисы
``` bash
sudo systemctl enable --now zabbix-server zabbix-agent nginx php8.1-fpm
```

### Перезапускаем nginx
``` bash
sudo systemctl restart nginx
```


### Проверка установки
После выполнения всех команд:
Откройте в браузере: http://Ваш_Ip_servera
Должен появиться веб-интерфейс Zabbix
Стандартные учетные данные:
- Логин: Admin
- Пароль: zabbix






Вот часть конфигурации Vagrantfile для развертывания Zabbix-сервера:

Если нету этой части кода её надо всатвить в `node.vm.provision "shell", inline: <<-SHELL`
``` shell
node.vm.provision "shell", inline: <<-SHELL
			
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

# Инструкция по использованию

Запуск виртуальной машины:

``` shell
vagrant up
```

Доступ к веб-интерфейсу Zabbix:

*Откройте в браузере: http://Ваш_Ip или http://localhost:8080*

Логин: Admin (с большой буквы)
Пароль: zabbix

Доступ по SSH:

``` shell
vagrant ssh
```

Основные команды управления сервисами (изнутри VM):

### Проверка статуса Zabbix
sudo systemctl status zabbix-server

### Перезапуск сервисов
sudo systemctl restart zabbix-server zabbix-agent postgresql
