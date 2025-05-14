# Инструкция по запуску проекта 

## 1. Установка необходимого ПО

### Для Windows:

1. Установите **VirtualBox**:  
    [Скачать с официального сайта](https://www.virtualbox.org/wiki/Downloads)
    
2. Установите **Vagrant**:  
	[Скачать с официального сайта](https://www.vagrantup.com/downloads)

	- Запустите скачанный `.exe` файл
	- Следуйте инструкциям установщика
	- Перезагрузите компьютер


### 1.2 Проверка установки

``` bash
vagrant --version
```

## 2 Создание ВМ

1. Перейдите в папку проекта:
2. Инициализируйте Vagrant:
``` bash
  vagrant init	
```
### 2.1 Настройка конфигурации

1. Замените содержимое `Vagrantfile`на:
	- vagrant/Vagrantfile
### 2.2 Запуск виртуальных машин

``` bash
  vagrant up
```
_После выполнения vagrant up у вас будут созданы 2 виртуальные машины `название машин most1 и most2`._

_Доступ к веб-интерфейсам после развертывания:_
- Zabbix-server
   - Адрес: http://<Ваш_IP_сервера> или http://localhost:80
   - Данные для входа:
     - Логин: Admin 
     - Пароль: zabbix


 - Jenkins
   - Адрес: http://<Ваш_IP_сервера>:8080
     
_Так как Jenkins запущен в Docker, выполните следующие команды:_
1. Подключитесь к виртуальной машине:
```bash
vagrant ssh most1
```

2. Найдите ID контейнера Jenkins:
```bash
sudo docker ps
```
3. Зайдите в контейнер:

```bash
sudo docker exec -it <ID_контейнера> bash
```
4. Просмотрите первоначальный пароль:
```bash
cat /var/jenkins_home/secrets/initialAdminPassword
```
- Подключение к машинам 

``` bash
vagrant ssh most1
```

``` bash
vagrant ssh most2
```

### 2.3 Работа с виртуальной машиной
- Подключиться по SSH:
  ``` bash
  vagrant ssh
  ```
- Приостановить работу:
    ``` bash
  vagrant suspend
    ```
- Полная остановка:
    ``` bash
  vagrant halt
    ```
- Удалить машину:
    
    ``` bash
  vagrant destroy
    ```

### 2.4 Перезагрузка конфигурации

После изменения Vagrantfile:
```
vagrant reload
```

#### Советы

1. При проблемах с запуском проверьте:
    
    - Включена ли виртуализация в BIOS
        
    - Достаточно ли свободного места на диске
        
    - Логи ошибок: `vagrant up --debug`
        
2. Для ускорения работы:
    
    - Используйте локальные образы (boxes)
        
        
3. Полезные команды:
    
    
    `vagrant status`    # Показать статус ВМ
   
    `vagrant port`      # Показать проброшенные порты
   
    `vagrant snapshot`  # Работа со снимками состояния

## 3 Создать проект уровня "hello world" на скриптоввом языке
Для реализации жтого проетка был выбран скриптовый язык такой как `bash`
``` bash
#!/bin/bash
Greetings="Hello, World!"
echo $Greetings
```

# 4 CI-часть:
# 5 CD-часть:

# 6 Zabbix:

