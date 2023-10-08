# Домашнее задание к занятию 1 «Введение в Ansible» - Соловцов Николай

## Подготовка к выполнению

1. Установите Ansible версии 2.10 или выше.
2. Создайте свой публичный репозиторий на GitHub с произвольным именем.
3. Скачайте [Playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.

## Основная часть

###1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте значение, которое имеет факт `some_fact` для указанного хоста при выполнении playbook.
####Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# ansible-playbook -i inventory/test.yml site.yml

PLAY [Print os facts] *******************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************
ok: [localhost]

TASK [Print OS] *************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***********************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP ******************************************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
###2. Найдите файл с переменными (group_vars), в котором задаётся найденное в первом пункте значение, и поменяйте его на `all default fact`.
####Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# cat group_vars/all/examp.yml
---
  some_fact: 'all default fact'
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# ansible-playbook -i inventory/test.yml site.yml

PLAY [Print os facts] *******************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************
ok: [localhost]

TASK [Print OS] *************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***********************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}

PLAY RECAP ******************************************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
###3. Воспользуйтесь подготовленным (используется `docker`) или создайте собственное окружение для проведения дальнейших испытаний.
####Ответ:
Подготовил docker-compose.yml и Dockerfile для ubuntu, что бы устанавливался python для работы ansible.
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# cat docker-compose.yml
version: '3.8'

services:

  debian:
    image: ubuntu
    container_name: ubuntu
    build:
      context: ubuntu
      dockerfile: Dockerfile
    networks:
      test_net:
        ipv4_address: 192.168.10.3
    tty: true

  centos7:
    image: centos:7
    container_name: centos7
    networks:
      test_net:
        ipv4_address: 192.168.10.2
    tty: true

networks:
  test_net:
    ipam:
      driver: default
      config:
        - subnet: 192.168.10.0/24
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# cat ubuntu/Dockerfile
FROM ubuntu:latest

RUN apt update && apt install -y python3
```
###4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из `managed host`.
####Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# ansible-playbook -i inventory/prod.yml site.yml

PLAY [Print os facts] *******************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***********************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP ******************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
###5. Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились значения: для `deb` — `deb default fact`, для `el` — `el default fact`.
####Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# cat group_vars/deb/examp.yml
---
  some_fact: "deb default fact"
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# cat group_vars/el/examp.yml
---
  some_fact: "el default fact"
```
###6.  Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.
####Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# ansible-playbook -i inventory/prod.yml site.yml

PLAY [Print os facts] *******************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***********************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP ******************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
###7. При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.
####Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# ansible-vault encrypt group_vars/deb/examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# ansible-vault encrypt group_vars/el/examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful
```
###8. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.
####Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# ansible-playbook -i inventory/prod.yml site.yml --ask-vault-password
Vault password:

PLAY [Print os facts] *******************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] *************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***********************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP ******************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
###9. Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.
####Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# ansible-doc -t shell -l
cmd        Windows Command Prompt
powershell Windows PowerShell
sh         POSIX shell (/bin/sh)
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# ansible-doc -t connection -l
community.docker.docker     Run tasks in docker containers
community.docker.docker_api Run tasks in docker containers
community.docker.nsenter    execute on host running controller container
local                       execute on controller
paramiko_ssh                Run tasks via python ssh (paramiko)
psrp                        Run tasks over Microsoft PowerShell Remoting Protocol
ssh                         connect via SSH client binary
winrm                       Run tasks over Microsoft's WinRM
```
Подходит модуль local.

###10. В `prod.yml` добавьте новую группу хостов с именем  `local`, в ней разместите localhost с необходимым типом подключения.
####Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# cat inventory/prod.yml
---
  el:
    hosts:
      centos7:
        ansible_connection: docker
  deb:
    hosts:
      ubuntu:
        ansible_connection: docker
  local:
    hosts:
      localhost:
        ansible_connection: local
```
###11. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь, что факты `some_fact` для каждого из хостов определены из верных `group_vars`.
####Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-01-base/playbook# ansible-playbook -i inventory/prod.yml site.yml --ask-vault-password
Vault password:

PLAY [Print os facts] *******************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************
ok: [ubuntu]
ok: [localhost]
ok: [centos7]

TASK [Print OS] *************************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ***********************************************************************************************************************************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}
ok: [localhost] => {
    "msg": "all default fact"
}

PLAY RECAP ******************************************************************************************************************************************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
12. Заполните `README.md` ответами на вопросы. Сделайте `git push` в ветку `master`. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым `playbook` и заполненным `README.md`.

## Необязательная часть

1. При помощи `ansible-vault` расшифруйте все зашифрованные файлы с переменными.
2. Зашифруйте отдельное значение `PaSSw0rd` для переменной `some_fact` паролем `netology`. Добавьте полученное значение в `group_vars/all/exmp.yml`.
3. Запустите `playbook`, убедитесь, что для нужных хостов применился новый `fact`.
4. Добавьте новую группу хостов `fedora`, самостоятельно придумайте для неё переменную. В качестве образа можно использовать [этот вариант](https://hub.docker.com/r/pycontribs/fedora).
5. Напишите скрипт на bash: автоматизируйте поднятие необходимых контейнеров, запуск ansible-playbook и остановку контейнеров.
6. Все изменения должны быть зафиксированы и отправлены в ваш личный репозиторий.

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---
