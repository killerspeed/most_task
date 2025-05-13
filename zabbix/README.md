# Установка и настройка Zabbix

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
