## Самоконтроль выполненения задания

>1. Где расположен файл с `some_fact` из второго пункта задания?

group_vars\all

>2. Какая команда нужна для запуска вашего `playbook` на окружении `test.yml`?

```commandline
ansible-playbook site.yml -i inventory/test.yml
```

>3. Какой командой можно зашифровать файл?

```commandline
ansible-vault encrypt <filename>
```   

>4. Какой командой можно расшифровать файл?

```commandline
ansible-vault decrypt <filename>
```  

>5. Можно ли посмотреть содержимое зашифрованного файла без команды расшифровки файла? Если можно, то как?

Можно. 
```commandline
ansible-vault view <filename>
```  

>6. Как выглядит команда запуска `playbook`, если переменные зашифрованы?
   
В этом случае необходимо добавить --ask-vault-pass к команде ansible-playbook.

```commandline
ansible-playbook site.yml -i inventory/prod.yml --ask-vault-pass
```

>7. Как называется модуль подключения к host на windows?

winrm

>8. Приведите полный текст команды для поиска информации в документации ansible для модуля подключений ssh

```commandline
ansible-doc -t connection ssh
```   

>9. Какой параметр из модуля подключения `ssh` необходим для того, чтобы определить пользователя, под которым необходимо совершать подключение?

remote_user