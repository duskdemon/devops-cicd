#devops-netology
### 09-ci-03-cicd
# Домашнее задание к занятию "09.03 CI\CD"

## Подготовка к выполнению

1. Создаём 2 VM в yandex cloud со следующими параметрами: 2CPU 4RAM Centos7(остальное по минимальным требованиям)
2. Прописываем в [inventory](./infrastructure/inventory/cicd/hosts.yml) [playbook'a](./infrastructure/site.yml) созданные хосты
3. Добавляем в [files](./infrastructure/files/) файл со своим публичным ключом (id_rsa.pub). Если ключ называется иначе - найдите таску в плейбуке, которая использует id_rsa.pub имя и исправьте на своё
4. Запускаем playbook, ожидаем успешного завершения

**Решение:**  
Запускаем playbook: "ansible-playbook site.yml -i inventory/cicd/hosts.yml"  
ошибка:  
```
TASK [Move Sonar into place.] *******************************************************************************************************
fatal: [sonar-01]: FAILED! => {"changed": false, "msg": "Destination directory /usr/local/sonar does not exist"}
```
исправляем, добавив таску в site.yml:  
```
    - name: Ensure installation dir exists
      become: true
      file:
        state: directory
        path: /usr/local/sonar
        mode: 0755
```
5. Проверяем готовность Sonarqube через [браузер](http://localhost:9000)
6. Заходим под admin\admin, меняем пароль на свой
7.  Проверяем готовность Nexus через [бразуер](http://localhost:8081)
8. Подключаемся под admin\admin123, меняем пароль, сохраняем анонимный доступ

## Знакомоство с SonarQube

### Основная часть

1. Создаём новый проект, название произвольное  
**Решение:** "dn-sonar, token dn-sonar: 387958913273d116228a5365ff45a67f69f2030f"
2. Скачиваем пакет sonar-scanner, который нам предлагает скачать сам sonarqube  
**Решение:** "sudo wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.7.0.2747-linux.zip"
3. Делаем так, чтобы binary был доступен через вызов в shell (или меняем переменную PATH или любой другой удобный вам способ)  
**Решение:** "export PATH=$PATH:/opt/sonar/sonar-scanner-4.7.0.2747-linux/bin"
4. Проверяем `sonar-scanner --version`
5. Запускаем анализатор против кода из директории [example](./example) с дополнительным ключом `-Dsonar.coverage.exclusions=fail.py`
```
sonar-scanner \
  -Dsonar.projectKey=dn-sonar \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://51.250.98.51:9000 \
  -Dsonar.login=387958913273d116228a5365ff45a67f69f2030f
  -Dsonar.coverage.exclusions=fail.py
```
6. Смотрим результат в интерфейсе
7. Исправляем ошибки, которые он выявил(включая warnings)  
Исправляем в fail.py :  
```
def increment(index):
    loc_index += 0
    loc_index += 1
    loc_index += index
    return loc_index
def get_square(numb):
    return numb*numb
def print_numb(numb):
    print("Number is {}".format(numb))

index = 0
while (index < 10):
    index = increment(index)
    print(get_square(index))
```
8. Запускаем анализатор повторно - проверяем, что QG пройдены успешно
9. Делаем скриншот успешного прохождения анализа, прикладываем к решению ДЗ  
**Решение:**  
![Скрин с ошибками](https://github.com/duskdemon/devops-netology-cicd/blob/main/sonar01.png)  
![Скрин после исправления](https://github.com/duskdemon/devops-netology-cicd/blob/main/sonar02.png)  

## Знакомство с Nexus

### Основная часть

1. В репозиторий `maven-public` загружаем артефакт с GAV параметрами:
   1. groupId: netology
   2. artifactId: java
   3. version: 8_282
   4. classifier: distrib
   5. type: tar.gz
2. В него же загружаем такой же артефакт, но с version: 8_102
3. Проверяем, что все файлы загрузились успешно
4. В ответе присылаем файл `maven-metadata.xml` для этого артефекта  
**Решение:**  
примечание: версии указал 312 и 322  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<metadata modelVersion="1.1.0">
<groupId>netology</groupId>
<artifactId>java</artifactId>
<versioning>
<latest>322</latest>
<release>322</release>
<versions>
<version>312</version>
<version>322</version>
</versions>
<lastUpdated>20220327144851</lastUpdated>
</versioning>
</metadata>
```
https://github.com/duskdemon/devops-netology-cicd/blob/main/maven-metadata.xml  

### Знакомство с Maven

### Подготовка к выполнению

1. Скачиваем дистрибутив с [maven](https://maven.apache.org/download.cgi)
2. Разархивируем, делаем так, чтобы binary был доступен через вызов в shell (или меняем переменную PATH или любой другой удобный вам способ)
3. Удаляем из `apache-maven-<version>/conf/settings.xml` упоминание о правиле, отвергающем http соединение( раздел mirrors->id: my-repository-http-unblocker)
4. Проверяем `mvn --version`
5. Забираем директорию [mvn](./mvn) с pom

### Основная часть

1. Меняем в `pom.xml` блок с зависимостями под наш артефакт из первого пункта задания для Nexus (java с версией 8_282)
2. Запускаем команду `mvn package` в директории с `pom.xml`, ожидаем успешного окончания
3. Проверяем директорию `~/.m2/repository/`, находим наш артефакт  
**Решение:** [артефакт в папке](https://github.com/duskdemon/devops-netology-cicd/blob/main/maven01.png)
4. В ответе присылаем исправленный файл `pom.xml`  
**Решение:**  
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.netology.app</groupId>
  <artifactId>simple-app</artifactId>
  <version>1.0-SNAPSHOT</version>
   <repositories>
    <repository>
      <id>my-repo</id>
      <name>maven-public</name>
      <url>http://51.250.103.144:8081/repository/maven-public/</url>
    </repository>
  </repositories>
  <dependencies>
    <dependency>
      <groupId>netology</groupId>
      <artifactId>java</artifactId>
      <version>322</version>
      <classifier>distrib</classifier>
      <type>tar.gz</type>
    </dependency>
  </dependencies>
</project>
```
https://github.com/duskdemon/devops-netology-cicd/blob/main/pom.xml  
