# Домашнее задание к занятию "`Ansible. Часть 2`" - `Долинин Илья`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. В личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1

1. Скачивание и распаковка архива <br>
Сценарий создает папку и скачивает архив Apache Kafka с официального сайта.

```yaml
---
- name: Download and unpack archive
  hosts: all
  become: yes
  vars:
    download_url: "https://www.apache.org/dyn/closer.lua/kafka/4.3.1/kafka-4.3.1-src.tgz?action=download"
    extract_dir: "/opt/kafka"
    archive_dest: "/tmp/kafka.tgz"

  tasks:
    - name: Create extraction directory
      ansible.builtin.file:
        path: "{{ extract_dir }}"
        state: directory
        mode: '0755'

    - name: Download the archive
      ansible.builtin.get_url:
        url: "{{ download_url }}"
        dest: "{{ archive_dest }}"
        mode: '0644'

    - name: Install tar
      ansible.builtin.package:
        name: tar
        state: present

    - name: Unpack the archive
      ansible.builtin.unarchive:
        src: "{{ archive_dest }}"
        dest: "{{ extract_dir }}"
        remote_src: yes
        extra_opts: [--strip-components=1]

```

`При необходимости прикрепитe сюда скриншоты
![Название скриншота 1](ссылка на скриншот 1)`

2. Установка и запуск tuned <br>
Сценарий ставит пакет, запускает службу и добавляет ее в автозагрузку.

```yaml
---
- name: Установка и настройка службы tuned
  hosts: all
  become: yes
  tasks:
    - name: Установить пакет tuned
      ansible.builtin.package:
        name: tuned
        state: present

    - name: Запустить tuned и добавить в автозагрузку
      ansible.builtin.service:
        name: tuned
        state: started
        enabled: yes

```

3. Изменение приветствия системы (motd) <br>
Сценарий задает новое приветствие, используя системную переменную.

```yaml
- name: Изменение приветствия системы (motd)
  hosts: all
  become: yes
  vars:
    new_motd: "Добро пожаловать на наш защищенный сервер!\n"
  tasks:
    - name: Записать новое приветствие в файл
      ansible.builtin.copy:
        content: "{{ new_motd }}"
        dest: /etc/motd
        mode: '0644'

```

<img width="1855" height="845" alt="Screenshot_11" src="https://github.com/user-attachments/assets/7d52c3d9-ab68-466f-b874-f6495969b2a4" />

<img width="837" height="805" alt="Screenshot_1" src="https://github.com/user-attachments/assets/0814ea23-d6cc-43f2-8789-6ae5cbcfadcd" />

Рекомендация: Для запуска сценариев каждый блок кода сохранён в отдельный файл с расширением .yml (playbook1.yml, playbook2.yml, playbook3.yml). 
Запустить их можно командой вида: 
```
ansible-playbook -i inventory.ini playbook1.yml
```

Файл inventory.ini:
```
[servers]
yc_vm ansible_host=<IP-адрес ВМ> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

---

### Задание 2

Для модификации плейбука были использованы встроенные переменные Ansible, такие как inventory_hostname (или ansible_hostname) для получения имени хоста и ansible_default_ipv4.address для IP-адреса.

```yaml
---
- name: Изменение приветствия системы (motd)
  hosts: all
  become: yes
  tasks:
    - name: Записать динамическое приветствие в файл
      ansible.builtin.copy:
        content: |
          Имя хоста: {{ ansible_hostname }}
          IP-адрес: {{ ansible_default_ipv4.address }}
          Добро пожаловать на наш защищенный сервер!
          Желаем отличного дня и продуктивной работы, системный администратор!

        dest: /etc/motd
        mode: '0644'

```

<img width="1848" height="581" alt="Screenshot_2" src="https://github.com/user-attachments/assets/fb146fa3-12ff-4113-9a02-41968f987e94" />

<img width="944" height="694" alt="Screenshot_3" src="https://github.com/user-attachments/assets/ae192de4-be63-45e4-a3a4-a1c0c34067db" />

Примечание: Переменная ansible_default_ipv4.address вернет IP-адрес сетевого интерфейса по умолчанию на целевой машине. На управляемых хостах должен быть установлен и корректно настроен Python для сбора фактов (Gathering Facts), иначе для получения данных о сети может потребоваться использование модуля setup перед выполнением блока copy.

---

### Задание 3

Роль site.yml:
```yaml
---
- name: Развертывание и настройка веб-сервера
  hosts: servers
  become: true
  roles:
    - web_server

```

Обработчик для переазпуска сервиса Apache:
```yaml
---
- name: Перезапуск Apache
  ansible.builtin.service:
    name: "{{ web_service_name }}"
    state: restarted

```

<img width="1858" height="866" alt="Screenshot_5" src="https://github.com/user-attachments/assets/15f5e64c-98e1-4526-8360-70bd90055d99" />
