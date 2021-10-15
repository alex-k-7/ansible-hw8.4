###Kibana-role
 
Роль состоит из следующих заданий:

precheck.yml:
```yaml
---
- name: Fail if unsupported system detected
  fail:
    msg: "System {{ ansible_distribution }} is not support by this role"
  when: ansible_distribution not in supported_systems
```

Проверяет поддерживаемые ОС, которые указаны в переменной [supported_systems].
Переменная указана в файле main.yml каталога vars:
```yaml
---
supported_systems: ['CentOS', 'Red Hat Enterprise Linux', 'Ubuntu', 'Debian']
```
---
download_apt.yml:
```yaml
---
- name: Download Kibana's deb
  get_url:
    url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kb_version }}-amd64.deb"
    dest: "/tmp/kibana-{{ kb_version }}-amd64.deb"
  register: download_kb
  until: download_kb is succeeded

```
Скачивает deb пакет Kibana в директорию tmp целевой машины (делает 3 попытки). Данная таска используется, если ОС целевой машины Debian/Ubuntu. 
Версия приложения указана в переменной [kb_version], находящейся в файле main.yml директории defaults:
```yaml
---
kb_version: "{{ elk_stack_version }}" # переменная elk_stack_version находится в group_vars
```
---
download_yum.yml:

Всё аналагично предыдущему заданию, но для ОС CentOS/RedHat.

---
install_yum.yml:
```yaml
---
- name: Install Kibana
  become: true
  yum:
    name: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"
    state: present
  notify: restart Kibana
```
Установка Kibana для ОС CentOS/RedHat с вызовом хэндлера перезагрузки сервиса kibana.
handlers/main.yml:
```yaml
---
- name: restart Kibana
  become: true
  systemd:
    name: kibana
    state: restarted
    enabled: true

```
---
install_apt.yml:

Всё аналагично предыдущему заданию, но для ОС Debian/Ubuntu.

---
configure.yml:
```yaml
---
- name: Configure Kibana
  become: true
  template:
    src: kibana.yml.j2
    dest: /etc/kibana/kibana.yml
  notify: restart Kibana
```
Копирует конфиг файл kibana из директории templates роли на целевую машину.

---
main.yml:

Файл описывающий выполняемые задания из директории tasks. Указана переменная из собранных фактов. 
Определяет пакетный менеджер ОС и в зависимости от этого запускается либо задание download_apt.yml, либо
download_yum.yml. То же самое для задания установки пакета.
```yaml
---
- import_tasks: precheck.yml  # 
- include_tasks: "download_{{ ansible_facts.pkg_mgr }}.yml"
- include_tasks: "install_{{ ansible_facts.pkg_mgr }}.yml" 
- import_tasks: configure.yml
```
---

Filebeat-role идентична kibana-role.