# Домашнее задание к занятию "09.04 Jenkins"

-----------------------------------------------------------------------------------------------------

* Декларативный [`Jenkinsfile`](https://github.com/lereklerik/ansible_test_roles/blob/jenkins/jenkins_ci/Jenkinsfile) в другом репозитории (ветка jenkins).
* Скриптовый [`ScriptedJenkinsFile`](https://github.com/lereklerik/ansible_role/blob/jenkins/jenkins_ci/Jenkinsfile) в другом репозитории (ветка jenkins).

-----------------------------------------------------------------------------------------------------

## Подготовка к выполнению

#### Создать 2 VM: для jenkins-master и jenkins-agent.

![vm's](pictures/01.png)

* `hosts.yml`:

```yaml
---
all:
  hosts:
    jenkins-master-01:
      ansible_host: 62.84.122.255
    jenkins-agent-01:
      ansible_host: 62.84.123.93
  children:
    jenkins:
      children:
        jenkins_masters:
          hosts:
            jenkins-master-01:
        jenkins_agents:
          hosts:
              jenkins-agent-01:
  vars:
    ansible_connection_type: paramiko
    ansible_user: netology
```
#### Установить jenkins при помощи playbook'a.

* [Лог](logs/00.md) запуска

#### Запустить и проверить работоспособность.

* `master`:

![hellomaster](pictures/02.png)

```shell
netology@netology:~$ ssh 62.84.122.255
[netology@centos-01 ~]$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
ede77a170b9f44f49c7**************de0c0
```

#### Сделать первоначальную настройку.

![hellomaster_03](pictures/03.png)

![hellomaster_04](pictures/04.png)

![hellomaster_05](pictures/05.png)

![hellomaster_06](pictures/06.png)

-----------------------------------------------------------------------------------------------------

## Основная часть

#### 1. Сделать Freestyle Job, который будет запускать molecule test из любого вашего репозитория с ролью.

* [Лог](logs/01.md) сборки

![job_01](pictures/07.png)

![job_01](pictures/08.png)

![job_01](pictures/09.png)

-----------------------------------------------------------------------------------------------------

#### 2. Сделать Declarative Pipeline Job, который будет запускать molecule test из любого вашего репозитория с ролью.
#### 3. Перенести Declarative Pipeline в репозиторий в файл Jenkinsfile.

* [`Jenkinsfile`](playbook/pipeline/Jenkinsfile) локально в этом репозитории.
* [`Jenkinsfile`](https://github.com/lereklerik/ansible_test_roles/blob/jenkins/jenkins_ci/Jenkinsfile) в другом репозитории.

```groovy
pipeline { 
    agent {
        label 'docker'
    } 
    stages {
        stage('Structure of directories'){
            steps {
                sh 'ls -l \$PWD'
            }
        }
        stage('Molecule test'){
            steps {
                sh 'cd playbook/roles/elasticsearch-role && molecule test'
            }                        
        }                
    }
}
```

![declarative](pictures/10.png)

* [Лог](logs/02.md) сборки

![dashboard_02](pictures/11.png)

-----------------------------------------------------------------------------------------------------

#### 4. Создать Multibranch Pipeline на запуск Jenkinsfile из репозитория.

![multibranch](pictures/13.png)

![multiscan](pictures/12.png)

* [Лог](logs/03.md) сканирования

![multidashboard](pictures/14.png)

![multijenkins](pictures/15.png)

![dashboard_03](pictures/16.png)

-----------------------------------------------------------------------------------------------------

#### 5. Создать Scripted Pipeline, наполнить его скриптом из pipeline.

* Скрипт:

```groovy
node ('docker'){
    stage('Checkout git') {
        git branch: 'jenkins', credentialsId: 'github', url: 'git@github.com:lereklerik/ansible_test_roles.git'
    }
    stage('Structure of directories'){
        sh 'ls -l \$PWD'
    }
    stage('Molecule test'){
        sh 'cd playbook/roles/elasticsearch-role && molecule test'
    } 
}
```
![scripted](pictures/17.png)

* [Лог](logs/04.md) сборки

![scripted_pipe](pictures/18.png)

-----------------------------------------------------------------------------------------------------

#### 6. Внести необходимые изменения, чтобы Pipeline запускал ansible-playbook без флагов --check --diff, если не установлен параметр при запуске джобы (prod_run = True), по умолчанию параметр имеет значение False и запускает прогон с флагами --check --diff.


* [`ScriptedJenkinsFile`](playbook/pipeline/ScriptedJenkinsFile) локально в этом репозитории.
* [`ScriptedJenkinsFile`](https://github.com/lereklerik/ansible_role/blob/jenkins/jenkins_ci/Jenkinsfile) в другом репозитории.

* Сам скрипт:

```groovy
node ('docker'){
    properties(
        [
            parameters(
                [string(defaultValue: 'False', name: 'prod_run')]
                )
        ]
    )
    stage('Checkout git') {
        git branch: 'jenkins', credentialsId: 'github', url: 'git@github.com:lereklerik/ansible_role.git'
    }
    stage('Ansible playbook'){
        if (prod_run.equalsIgnoreCase("True")) {
            sh 'ansible-playbook -i inventory/elk/hosts.yml site.yml --check --diff'
        } else {
            sh 'ansible-playbook -i inventory/elk/hosts.yml site.yml'           

        }
    } 
}
```

![elk_01](pictures/19.png)

-----------------------------------------------------------------------------------------------------

#### 7. Проверить работоспособность, исправить ошибки, исправленный Pipeline вложить в репозиторий в файл ScriptedJenkinsfile. Цель: получить собранный стек ELK в Ya.Cloud.

* Попыток собрать нормальный стек было много. В основном были какие-то внутренние ошибки в части отсутствия доступа к директории, невозможности отработать задачу без прав root-пользователя, отсутствия папки `files` для складирования дистрибутивов (тут как раз было больше всего пробных запусков).
* Собрала стек на агенте, т.к. попытки связать агента с другими ВМ в яндексе были тоже неудачными.

![elk_02](pictures/20.png)

![elk_03](pictures/21.png)

* [Лог](logs/05.md) запуска с `prod_run = "False"`

* [Лог](logs/06.md) запуска с `prod_run = "True"`

![elk_04](pictures/elk_01.png)

![elk_04](pictures/elk_02.png)

* Dashboard в Jenkins:

![jenkinsfinal](pictures/22.png)
