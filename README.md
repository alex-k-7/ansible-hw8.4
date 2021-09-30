```yaml
- name: Install Kibana #имя плейбука  
  hosts: elk #хосты на которых выполнять таски
  tasks:
    - name: Upload tar.gz Kibana from remote URL
      get_url: #модуль скачивает файл по url
        url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz" #куда положить скаченный файл 
        mode: 0755
        timeout: 60
        force: true
        validate_certs: false
      register: get_kibana
      until: get_kibana is succeeded
      tags: kibana
    - name: Create directrory for Kibana
      file:
        state: directory
        path: "{{ kibana_home }}"
      tags: kibana
    - name: Extract Kibana in the installation directory
      become: true
      unarchive:
        copy: false
        src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "{{ kibana_home }}"
        extra_opts: [--strip-components=1]
        creates: "{{ kibana_home }}/bin/kibana"
      tags:
        - kibana
    - name: Set environment Kibana
      become: true
      template:
        src: templates/kibana.sh.j2
        dest: /etc/profile.d/kibana.sh
      tags: kibana
```