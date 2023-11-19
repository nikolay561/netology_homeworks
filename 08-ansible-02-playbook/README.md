# Домашнее задание к занятию 2 «Работа с Playbook» - Соловцов Николай

## Подготовка к выполнению

1. * Необязательно. Изучите, что такое [ClickHouse](https://www.youtube.com/watch?v=fjTNS2zkeBs) и [Vector](https://www.youtube.com/watch?v=CgEhyffisLY).
2. Создайте свой публичный репозиторий на GitHub с произвольным именем или используйте старый.
3. Скачайте [Playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
4. Подготовьте хосты в соответствии с группами из предподготовленного playbook.

## Основная часть

1. Подготовьте свой inventory-файл `prod.yml`.
Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-02-playbook/playbook# cat inventory/prod.yml
---
clickhouse:
  hosts:
    clickhouse:
      ansible_connection: docker

vector:
  hosts:
    vector:
      ansible_connection: docker
```
2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает [vector](https://vector.dev). Конфигурация vector должна деплоиться через template файл jinja2. От вас не требуется использовать все возможности шаблонизатора, просто вставьте стандартный конфиг в template файл. Информация по шаблонам по [ссылке](https://www.dmosk.ru/instruktions.php?object=ansible-nginx-install).\
Ответ:
```
- name: Install Vector
  hosts: vector
  handlers:
    - name: Start vector service
      become: true
      ansible.builtin.service:
        name: vector
        state: restarted
  tasks:
    - name: Get vector distrib
      ansible.builtin.get_url:
        url: "https://packages.timber.io/vector/{{ vector_version }}/vector-{{ vector_version }}-1.x86_64.rpm"
        dest: ./vector_{{ vector_version }}.rpm
        mode: 0755
    - name: Instal vector
      become: true
      ansible.builtin.yum:
        disable_gpg_check: true
        name: ./vector_{{ vector_version }}.rpm
```

Переменная vector_version хранится в playbook/group_vars/vector/vars.yml:
```

---
vector_version: "0.33.0"
```

3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
Ответ: Использован модуль ansible.builtin.get_url для загрузки необходимого дистрибутива.

4. Tasks должны: скачать дистрибутив нужной версии, выполнить распаковку в выбранную директорию, установить vector.
Ответ:
Скачивается дистрибутив vector версии 0.33.0, распаковка не требуется, пакет устанавливается через менеджер пакетов yum, используется модуль ansible.builtin.yum.

5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
Ответ: ansible-lint site.yml ошибок не показывает.

6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-02-playbook/playbook# ansible-playbook -i inventory/prod.yml site.yml --check

PLAY [Install Clickhouse] ***************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ***********************************************************************************************************************************************************************
changed: [clickhouse-01] => (item=clickhouse-client)
changed: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ***********************************************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Install clickhouse packages] ******************************************************************************************************************************************************************
fatal: [clickhouse-01]: FAILED! => {"changed": false, "msg": "No RPM file matching 'clickhouse-common-static-22.3.3.44.rpm' found on system", "rc": 127, "results": ["No RPM file matching 'clickhouse-common-static-22.3.3.44.rpm' found on system"]}

PLAY RECAP ******************************************************************************************************************************************************************************************
clickhouse-01              : ok=2    changed=1    unreachable=0    failed=1    skipped=0    rescued=1    ignored=0

root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-02-playbook/playbook#
```

Команда ansible-playbook -i inventory/prod.yml site.yml --check показывает ошибку, так как не может найти дистрибутив, который скачивается по url. Не нашел как это побороть, работе плейбука это не мешает.

7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-02-playbook/playbook# ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Clickhouse] ***************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ***********************************************************************************************************************************************************************
changed: [clickhouse-01] => (item=clickhouse-client)
changed: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "item": "clickhouse-common-static", "msg": "Request failed", "response": "HTTP Error 404: Not Found", "status_code": 404, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ***********************************************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Install clickhouse packages] ******************************************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Flush handlers] *******************************************************************************************************************************************************************************

RUNNING HANDLER [Start clickhouse service] **********************************************************************************************************************************************************
changed: [clickhouse-01]

TASK [Create database] ******************************************************************************************************************************************************************************
changed: [clickhouse-01]

PLAY [Install Vector] *******************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************
ok: [vector-01]

TASK [Get vector distrib] ***************************************************************************************************************************************************************************
changed: [vector-01]

TASK [Instal vector] ********************************************************************************************************************************************************************************
changed: [vector-01]

PLAY RECAP ******************************************************************************************************************************************************************************************
clickhouse-01              : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
vector-01                  : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-02-playbook/playbook#
```

8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
Ответ:
```
root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-02-playbook/playbook# ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Clickhouse] ***************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Get clickhouse distrib] ***********************************************************************************************************************************************************************
ok: [clickhouse-01] => (item=clickhouse-client)
ok: [clickhouse-01] => (item=clickhouse-server)
failed: [clickhouse-01] (item=clickhouse-common-static) => {"ansible_loop_var": "item", "changed": false, "dest": "./clickhouse-common-static-22.3.3.44.rpm", "elapsed": 0, "gid": 0, "group": "root", "item": "clickhouse-common-static", "mode": "0644", "msg": "Request failed", "owner": "root", "response": "HTTP Error 404: Not Found", "size": 246310036, "state": "file", "status_code": 404, "uid": 0, "url": "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-22.3.3.44.noarch.rpm"}

TASK [Get clickhouse distrib] ***********************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Install clickhouse packages] ******************************************************************************************************************************************************************
ok: [clickhouse-01]

TASK [Flush handlers] *******************************************************************************************************************************************************************************

TASK [Create database] ******************************************************************************************************************************************************************************
ok: [clickhouse-01]

PLAY [Install Vector] *******************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************
ok: [vector-01]

TASK [Get vector distrib] ***************************************************************************************************************************************************************************
ok: [vector-01]

TASK [Instal vector] ********************************************************************************************************************************************************************************
ok: [vector-01]

PLAY RECAP ******************************************************************************************************************************************************************************************
clickhouse-01              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=1    ignored=0
vector-01                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

root@my-server:/home/solovtsov/homework/netology_homeworks/08-ansible-02-playbook/playbook#
```

9. Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги. Пример качественной документации ansible playbook по [ссылке](https://github.com/opensearch-project/ansible-playbook).
Ответ:

Устанавливается Clickhouse, после установки сервис перезагружается.
```
---
- name: Install Clickhouse
  hosts: clickhouse
  handlers:
    - name: Start clickhouse service
      become: true
      ansible.builtin.service:
        name: clickhouse-server
        state: restarted
```
Пакеты для установки Clickhouse скачиваются с https://packages.clickhouse.com/, в блоке rescue предусмотрена запасная ссылка на дистрибутив, если первая ссылка будет не доступна. 
```
  tasks:
    - block:
        - name: Get clickhouse distrib
          ansible.builtin.get_url:
            url: "https://packages.clickhouse.com/rpm/stable/{{ item }}-{{ clickhouse_version }}.noarch.rpm"
            dest: "./{{ item }}-{{ clickhouse_version }}.rpm"
          with_items: "{{ clickhouse_packages }}"
      rescue:
        - name: Get clickhouse distrib
          ansible.builtin.get_url:
            url: "https://packages.clickhouse.com/rpm/stable/clickhouse-common-static-{{ clickhouse_version }}.x86_64.rpm"
            dest: "./clickhouse-common-static-{{ clickhouse_version }}.rpm"
    - name: Install clickhouse packages
      become: true
      ansible.builtin.yum:
        name:
          - clickhouse-common-static-{{ clickhouse_version }}.rpm
          - clickhouse-client-{{ clickhouse_version }}.rpm
          - clickhouse-server-{{ clickhouse_version }}.rpm
      notify: Start clickhouse service
    - name: Flush handlers
      meta: flush_handlers
    - name: Create database
      ansible.builtin.command: "clickhouse-client -q 'create database logs;'"
      register: create_db
      failed_when: create_db.rc != 0 and create_db.rc !=82
      changed_when: create_db.rc == 0
```
Устанавливается Vector, после установки сервис перезагружается.
```
- name: Install Vector
  hosts: vector
  handlers:
    - name: Start vector service
      become: true
      ansible.builtin.service:
        name: vector
        state: restarted
```
Пакет для установки Clickhouse скачивается с https://packages.clickhouse.com/.
```
  tasks:
    - name: Get vector distrib
      ansible.builtin.get_url:
        url: "https://packages.timber.io/vector/{{ vector_version }}/vector-{{ vector_version }}-1.x86_64.rpm"
        dest: ./vector_{{ vector_version }}.rpm
        mode: 0755
    - name: Instal vector
      become: true
      ansible.builtin.yum:
        disable_gpg_check: true
        name: ./vector_{{ vector_version }}.rpm
```

10. Готовый playbook выложите в свой репозиторий, поставьте тег `08-ansible-02-playbook` на фиксирующий коммит, в ответ предоставьте ссылку на него.

---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---
