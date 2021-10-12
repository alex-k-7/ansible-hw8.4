```yaml
---
- name: Install Elasticsearch             # Play установки и настройки Elasticsearch
  hosts: elasticsearch
  handlers:                               # хэндлер на перезапуск сервиса, также активирует автозапуск 
   - name: restart Elasticsearch
     become: true
     systemd:                         
       name: elasticsearch          
       state: restarted
       enabled: true
  tasks:                           
   - name: Download Elasticsearch's rpm 
     get_url:                             # скачивает пакет Elasticsearch в <dest>
       url: "https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"
       dest: "/tmp/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"
     register: download_elastic           # писать результаты выполнения get_url в переменную download_elastic
     until: download_elastic is succeeded # повторять get_url пока не выполнится удачно (но не более 3х раз)
   - name: Install Elasticsearch
     become: true
     yum:                                 # установка пакета <name>
       name: "/tmp/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"
       state: present
   - name: Configure Elasticsearch
     become: true                         # повышение привилегий
     template:                            # копирует файл конфигурации Elasticsearch из <src> в <dest>. при этом умеет интерполировать переменные
       src: elasticsearch.yml.j2
       dest: /etc/elasticsearch/elasticsearch.yml
     notify: restart Elasticsearch        # активация действия в handlers при наличии изменений в данной таске

- name: Install Kibana                    # Play установки и настройки Kibana (идентичен предыдущему)
  hosts: kibana
  handlers:
   - name: restart Kibana
     become: true
     systemd:
       name: kibana
       state: restarted
       enabled: true
  tasks:
   - name: Download Kibana's rpm
     get_url:
       url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ elk_stack_version }}-x86_64.rpm"
       dest: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"
     register: download_elastic
     until: download_elastic is succeeded
   - name: Install Kibana
     become: true
     yum:
       name: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"
       state: present
   - name: Configure Kibana
     become: true
     template:
       src: kibana.yml.j2
       dest: /etc/kibana/kibana.yml
     notify: restart Kibana

- name: Install Filebeat                  # Play установки и настройки Filebeat (идентичен предыдущему)
  hosts: filebeat
  handlers:
   - name: restart filebeat
     become: true
     systemd:
       name: filebeat
       state: restarted
       enabled: true
  tasks:
   - name: Download filebeat's rpm"
     get_url:
       url: "https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-{{ elk_stack_version }}-x86_64.rpm"
       dest: "/tmp/filebeat-{{ elk_stack_version }}-x86_64.rpm"
     register: download_fb
     until: download_fb is succeeded
   - name: Install Filebeat
     become: true
     yum:
       name: "/tmp/filebeat-{{ elk_stack_version }}-x86_64.rpm"
       state: present
   - name: Configure Filebeat
     become: true
     template:
       src: filebeat.yml.j2
       dest: /etc/filebeat/filebeat.yml
     notify: restart filebeat
   - name: Set Filebeat systemwork        # настройка filebeat на передачу в elasticsearch системных логов
     become: true
     command:                             # выполнение команды <cmd> из директории <chdir>       
       cmd: filebeat modules enable system
       chdir: /usr/share/filebeat/bin
   - name: Load Kibana dashboard          # установка дашбордов в kibana
     become: true
     command:
       cmd: filebeat setup
       chdir: /usr/share/filebeat/bin
     register: fb_setup
     until: fb_setup is succeeded
     notify: restart filebeat
```