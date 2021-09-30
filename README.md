```yaml
- name: Install Kibana #имя плейбука  
  hosts: elk #хосты на которых выполнять таски
  tasks:
    - name: Upload tar.gz Kibana from remote URL
      get_url: #модуль скачивает файл по url
        url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz" #куда положить скаченный файл 
        mode: 0755 #права на файл
        timeout: 60
        force: true
        validate_certs: false
      register: get_kibana #писать результаты выполнения get_url в переменную get_kibana
      until: get_kibana is succeeded #повторять get_url пока не выполнится удачно (но не более 3х раз)
      tags: kibana #тэг таски
    - name: Create directrory for Kibana
      file: #создает дирректорию
        state: directory
        path: "{{ kibana_home }}"
      tags: kibana
    - name: Extract Kibana in the installation directory
      become: true #повышение привелегий
      unarchive: #разархивироание
        copy: false #указывает, что архив находится не на хосте управления, а на целевом.
        src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz" #архив
        dest: "{{ kibana_home }}" #куда разархивировать
        extra_opts: [--strip-components=1] #доп опция для архиватора
        creates: "{{ kibana_home }}/bin/kibana" #проверка появился ли файл kibana
      tags:
        - kibana
    - name: Set environment Kibana
      become: true
      template: #модуль копирует файл src в dest. при этом умеет интерполировать переменные
        src: kibana.sh.j2
        dest: /etc/profile.d/kibana.sh
      tags: kibana
```