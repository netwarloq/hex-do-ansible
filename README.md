# Репозиторий прохождения интенсива "DevOps для программистов" на Хекслет

Данный репозиторий содержит самостоятельные работы курсанта по курсу Хекслета инструмента Ansible (система автоматизации, управления конфигурациями).\
*В будущем будет дополнен другими темами (инструментами), либо появятся отдельные репозитории.*

В ходе прохождения интенсива используются ВМ в DigitalOcean (можно использовать любой аналог), а также локальный хост.
Для удобства *ip/fqdn* ВМ прописаны в */etc/hosts* на машине с установленным ansible. Было заранее сгенерированна пара ключей для подключения по протоколу ssh и загружены публичные ключи на созданные лабораторные ВМ.
<hr>
Директории <i>mf_{name}</i> содержат файлы инструкций MakeFile некоторых заданий, где <i>{name}</i> - краткое имя темы задания.

Для запуска Makefile переходим в нужную директорию и выполняем команду make с указанием цели (например *start*):

```
cd mf_ping
make start
```
<hr>
Директория <i>inventory_file</i> содержит файл инвентаризации, в котором описан список хостов и определены необходимые переменные, например такие как:

```
ansible_user={пользователь ВМ}
ansible_ssh_private_key_file={путь до приватного ключа ssh}
```
<hr>
Директория <i>files</i> содержит вспомогательные файлы, шаблоны для выполнения конкретных задач. Также некоторые шаблоны (jinja2) находятся в директории <i>templates</i>.
<hr>
В директории <i>playbooks</i> находятся все файлы ansible playbook (файлы сценарии выполнения команд).
Наименования плейбуков по двум правилам:

1. <i>pb_action_item_{opt}</i>, где action - действие (например set: настройка, inst: установка, rem: удаление), item - объект, над которым проводится действие и {opt} - опционально, для уточнения действия или темы.
2. <i>pb_theme</i>, где theme - тема урока для которого был написан плейбук.\
    <i>pb_</i> - сокращение <b>p</b>lay<b>b</b>ook

<hr>

## Как пользоваться 
Подробную информация по инструментам ansible смотреть официальной документации, здесь будут приведены некоторые примеры используемые в ходе выполнения заданий.

- Для проверки файла инвентаризации существует инструмент для выгрузки `ansible-inventory`

    Пример:
    ```
    $ ansible-inventory -i inventory-file/hex.ini  --list
    ```
    <details><summary>Вывод команды</summary>
    
    ```
    $ ansible-inventory -i inventory-file/hex.ini  --list
    {
        "_meta": {
            "hostvars": {
                "hex-do1": {
                    "ansible_ssh_private_key_file": "~/.ssh/hexdo",
                    "ansible_user": "root",
                    "server_name": "websrv"
                },
                "hex-do2": {
                    "ansible_ssh_private_key_file": "~/.ssh/hexdo",
                    "ansible_user": "root",
                    "server_name": "appsrv"
                },
                "hex-do3-cent": {
                    "ansible_ssh_private_key_file": "~/.ssh/hexdo",
                    "ansible_user": "root",
                    "server_name": "centos-srv"
                },
                "hex-do4-deb": {
                    "ansible_ssh_private_key_file": "~/.ssh/hexdo",
                    "ansible_user": "root"
                }
            }
        },
        "all": {
            "children": [
                "allservs",
                "buntuhttp",
                "debcent",
                "servs",
                "ungrouped"
            ]
        },
        "allservs": {
            "children": [
                "appserv",
                "cents",
                "deb",
                "webserv"
            ]
        },
        "appserv": {
            "hosts": [
                "hex-do2"
            ]
        },
        "buntuhttp": {
            "children": [
                "appserv",
                "webserv"
            ]
        },
        "cents": {
            "hosts": [
                "hex-do3-cent"
            ]
        },
        "deb": {
            "hosts": [
                "hex-do4-deb"
            ]
        },
        "debcent": {
            "children": [
                "cents",
                "deb"
            ]
        },
        "servs": {
            "children": [
                "appserv",
                "webserv"
            ]
        },
        "webserv": {
            "hosts": [
                "hex-do1"
            ]
        }
    }
    ```
    </details>
    Ключ <i>-i</i> чтобы указать файл инвентаризации (.ini), <i>--list</i> для вывода всей информации о хостах.

- Запуск плейбука
    
    ```
    ansible-playbook playbooks/pb_set_nginx.yaml -i inventory-file/hex.ini --tags html --check
    ```
    Указываем необходимый плейбук, затем файл инвентаризации. Дополнительно можно указать тег задач, для того чтобы выполнить определенный список задач, с помощью ключа <i>--tags (либо -t)</i>. А также провести проверку возможности выполнения плейбука ключом <i>--check</i>