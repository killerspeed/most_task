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

## 4 CI-часть:
Для реализация удалённой сборки проекта (из п. №2) с помощью CI-pipeline был использован jenkins 
- Требования
	- Установленный и настроенный сервер Jenkins 
      - как устанвоить Jenkins можно посмотерть в репозитории устанвока [Jenkins](https://github.com/killerspeed/most_task/tree/version/jenkins)
 	- Доступ к репозиторию с исходным кодом проекта
- Настройка Jenkins
	- Установите необходимые плагины в Jenkins:
 		- Pipeline
   		-  Git

    
![img.png](img.png)


Для создание Pipeline нажимаем создание item:
- Выберите тип "Pipeline"
- Укажите название вашего проекта

![img_1.png](img/name.png)


  
## 5 CD-часть:

## 6 Zabbix:
### Установка и настройка Zabbix
*Установите Zabbix, следуя инструкции из репозитория:*
https://github.com/killerspeed/most_task/blob/version/zabbix/README.md

_Доступ к веб-интерфейсу:_
- После установки откройте веб-интерфейс Zabbix в браузере:
- Адрес: http://<IP_вашего_сервера> или http://localhost:80
- 
#### Добавление нового хоста в Zabbix

*Добавление второй машины для мониторинга:*
 - Перейдите в раздел Configuration → Hosts. 
 - Нажмите Create Host и заполните необходимые параметры.
_После нажатия Create Host откроется форма настройки хоста. Заполните следующие поля:_
- host name - задать имя хоста 
- Templates - Выберите подходящие шаблон 
- Groups - Назначьте хост в группу (например, Linux Servers)
- Interfaces type agent - Нажмите Add и укажите:
  - IP-адрес сервера.
  - Порт (по умолчанию 10050 для Zabbix Agent).

![add_vm.png](img%2Fadd_vm.png)


#### Завершение настройки хоста в Zabbix

1. Сохранение настроек
   - Нажмите кнопку Update для применения изменений.
2. Проверка подключения 
   - Если вы использовали мой Vagrantfile:
     - Хост должен отображаться в списке с зелёным индикатором (успешное подключение).

![img.png](img/img.png)
3. Ручная установка Zabbix Agent (если использовали свой сервер)
```bash
# Обновление системы
sudo apt update && sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt autoremove -y

# Установка репозитория Zabbix
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4+ubuntu$(lsb_release -rs)_all.deb
sudo dpkg -i zabbix-release_6.0-4+ubuntu$(lsb_release -rs)_all.deb
sudo apt update

# Установка и настройка Zabbix Agent
sudo apt install zabbix-agent -y

# Конфигурация агента
sudo sed -i 's/^ServerActive=127.0.0.1.*/ServerActive=IP_ZABBIX_SERVER/' /etc/zabbix/zabbix_agentd.conf
sudo sed -i 's/^Server=127.0.0.1.*/Server=IP_ZABBIX_SERVER/' /etc/zabbix/zabbix_agentd.conf
sudo sed -i 's/^Hostname=Zabbix server.*/Hostname=ИМЯ_ХОСТА_ИЗ_ZABBIX/' /etc/zabbix/zabbix_agentd.conf

# Перезапуск агента
sudo systemctl restart zabbix-agent.service
```


## Важные замечания
- Замените IP_ZABBIX_SERVER на IP-адрес вашего Zabbix-сервера.
- Укажите ИМЯ_ХОСТА_ИЗ_ZABBIX — имя, которое вы задали при создании хоста (поле Host name).

#### После добавления хоста и успешного подключения агента можно настроить отображение метрик на Dashboard.

1. Переход в Dashboard 
- В веб-интерфейсе Zabbix перейдите в раздел: `Monitoring` → `Dashboard` → `All dashboards`

2. Добавление виджетов
- Нажмите `Edit dashboard` (или `Create dashboard`, если он ещё не создан).

![img_1.png](img/img_1.png)
3. Настройка графика
Выберите `Graph` → `Add`.

_Укажите:_
- Название (например, "`CPU utilization`").
- `type` `Graph`

![img.png](img/img2.png)

_Далее укажем имя хоста в нашем случае `most2` и метрики `Linux: CPU user time` `Linux: Load average (1m avg)
`_

![img.png](img/img11.png)

все тоже самое делаем для мониторинга Диска и Памяти и так же для второй машины

4. Готовый дашборд
После добавления всех нужных виджетов:

*Нажмите Save dashboard.*

![img_1.png](img/img_11.png)


