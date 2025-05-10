# V1

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

# V2
1. **Исправленный и улучшенный вариант конфигурации Vagrant**

- *По итогу тестов было принято решено сразу устанволивтаь 22,04
Меняем строчку:*

`Было: bento/ubuntu-20.04`

`Стало: node.vm.box = "ubuntu/jammy64"`

# Install Jenkins
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