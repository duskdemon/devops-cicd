#devops-netology
### 09-ci-04-jenkins
# Домашнее задание к занятию "09.04 Jenkins"

## Подготовка к выполнению

1. Создать 2 VM: для jenkins-master и jenkins-agent.
2. Установить jenkins при помощи playbook'a.
3. Запустить и проверить работоспособность.
4. Сделать первоначальную настройку.

**Выполнение:**  
1. Создано 2 ВМ на ресурсах Яндекс клауд: мастер 51.250.103.9 и агент 84.252.143.225.  
2. После запуска плейбука столкнулся с ошибкой в процессе установки Ансибл(кодировка не совпадает), решил с помощью установки $LANG=en_US.UTF-8 и $LC_ALL=en_US.UTF-8.  
3. Заходим на мастер ноду по порту 8080 и вводим пароль, предварительно взяв его из ВМ мастера по пути /var/lib/jenkins/secrets/initialAdminPassword.
4. Проводим установку через Веб интерфейс "Install suggested plugins", jenkins доступен по "http://51.250.103.9:8080/". Делаем настройку сборщика на агенте, запускаем, смотрим лог:
```
$ ssh 84.252.143.225 java -jar /opt/jenkins_agent/agent.jar
<===[JENKINS REMOTING CAPACITY]===>channel started
Remoting version: 4.13
This is a Unix agent
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by jenkins.slaves.StandardOutputSwapper$ChannelSwapper to constructor java.io.FileDescriptor(int)
WARNING: Please consider reporting this to the maintainers of jenkins.slaves.StandardOutputSwapper$ChannelSwapper
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
Evacuated stdout
Agent successfully connected and online
```

## Основная часть

1. Сделать Freestyle Job, который будет запускать `molecule test` из любого вашего репозитория с ролью.

**Выполнение:**  
Создаем свободную задачу, добавляя в разделе гит в нее ссылку на репозиторий "git@github.com:duskdemon/filebeat-role.git"  и добавляем пользовательские учетные данные(персональный ключ гита). Задача завершается с ошибкой, т.к. продукты elastic.co заблокированы для ip-адресов в России(Яндекс клауд).
```
17:33:16 fatal: [ubuntu -> localhost]: FAILED! => {"attempts": 3, "changed": false, "dest": "files/kibana-7.14.0-amd64.deb", "elapsed": 0, "msg": "Request failed", "response": "HTTP Error 403: Forbidden", "status_code": 403, "url": "https://artifacts.elastic.co/downloads/kibana/kibana-7.14.0-amd64.deb"}
```
2. Сделать Declarative Pipeline Job, который будет запускать `molecule test` из любого вашего репозитория с ролью.
3. Перенести Declarative Pipeline в репозиторий в файл `Jenkinsfile`.
4. Создать Multibranch Pipeline на запуск `Jenkinsfile` из репозитория.
5. Создать Scripted Pipeline, наполнить его скриптом из [pipeline](./pipeline).
6. Внести необходимые изменения, чтобы Pipeline запускал `ansible-playbook` без флагов `--check --diff`, если не установлен параметр при запуске джобы (prod_run = True), по умолчанию параметр имеет значение False и запускает прогон с флагами `--check --diff`.
7. Проверить работоспособность, исправить ошибки, исправленный Pipeline вложить в репозиторий в файл `ScriptedJenkinsfile`. Цель: получить собранный стек ELK в Ya.Cloud.
8. Отправить две ссылки на репозитории в ответе: с ролью и Declarative Pipeline и c плейбукой и Scripted Pipeline.

**Выполнение:**  

Поскольку продукты elasticco заблокированы для ip-адресов в России, не получается собрать стек ELK, т.к. плейбуки выпадают с ошибкой  

Выполнение пунктов по созданию джобов и пайплайнов подтверждаю скриншотом:  
![скрин Дженкинса](https://github.com/duskdemon/devops-netology-cicd/blob/main/jenkins01.png)  
Ниже ссылки на дженкинс-файлы  
1. [Jenkinsfile](https://github.com/duskdemon/devops-netology-cicd/blob/main/Jenkinsfile)  
2. [ScriptedJenkinsfile](https://github.com/duskdemon/devops-netology-cicd/blob/main/ScriptedJenkinsfile)  

