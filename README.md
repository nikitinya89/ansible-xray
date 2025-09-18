# Описание
**Ansible** playbook для настройки *xray* сервера.

Можно выбрать между двумия типами безопасности: **reality** и **tls**.
- В первом случае маскируемся под реальный сайт.
- Во втором - поднимаем *nginx* сервер и получаем *Lets Encrypt* сертификаты для своего домена.
> В случае использования tls предварительно необходимо добавить A запись в DNS, указывающую на ваш сервер.

В обоих случаях создается отдельный конфиг config-tls.json или config-reality.json, на который делается симлинк config.json. Таким образом можно переключаться между двумя конфигурациями, переопределив значение переменной `security`.
> Конфигурационный файл создается только при первом запуске соотвествующего типа. При повторном происходит только переключение симлинка.

> При использовании *reality* публичный ключ для клиента записывается в файл `/opt/xray/reality_pub_key`
По умолчанию настраивается *xray reality* сервер, маскирующийся под `www.microsoft.com`. 


## Предварительная настройка

Необходимо добавить свой сервер в `inventory/hosts.ini`
```
my_server ansible_host=10.10.10.10 ansible_user=my_user
```
> *my_user* - пользователь на удаленном сервере, состоящий в группе **sudo**

Значения переменных по умолчанию можно переопределить:
- непосредственно в *cli* команде запуска плэйбука, используя ключ `-e` для каждой переменной, например `-e xray_version=1.1.1`
- указав их через пробел в `inventory/hosts.ini`, например `xray_version=1.1.1`
- в отдельном файле `inventory/host_vars/my_server.yml` (`my_server` совпадает с именем хоста в `inventory/hosts.ini`):
```
xray_version: 1.1.1
ansible_user: my_user
```

## Запуск
```
ansible-playbook xray.yml
```

Если для подключения по SSH вместо ключа используется пароль, добавляем ключ `--ask-pass`.
Если использование `sudo` требует указания пароля, добавляем ключ `--ask-become-pass`.
```
ansible-playbook xray.yml --ask-pass --ask-become-pass
```

В случае наличия нескольких серверов в интвентаре можно ограничить запуск плэйбука на конкретном сервере с помощью ключа `--limit`
```
ansible-playbook xray.yml --limit my_server
```

### Переменные 
| Переменная       | Тип      | Значение по умолчанию  | Описание                                                         |
| ---------------- | -------- | ---------------------- | ---------------------------------------------------------------- |
| **xray_version** | *string* | *25.9.11*              | Версия *Xray*                                                    |
| **xray_path**    | *string* | */opt/xray*            | Каталог для установки *Xray*                                     |
| **security**     | *bool*   | *reality*              | Тип безопасности *xray*: **reality** или **tls**                 |
| **reality_site** | *string* | *www.microsoft.com*    | Сайт для маскировки (при *security* **reality**)                 |
| **nginx_port**   | *string* | *8080*                 | Порт веб-сервера *nginx* для маскировки (при *security* **tls**) |
| **domain_name**  | *string* |                        | Доменное имя для создания сертификатов (при *security* **tls**). Необходимо, чтобы существовала А-запись для этого имени.  |

### Примеры

#### Reality сервер www.microsoft.com
```
ansible-playbook xray.yml --ask-pass --ask-become-pass
```
#### Reality сервер www.microsoft.com. Версия Xray v25.7.25
```
ansible-playbook xray.yml --ask-pass --ask-become-pass -e xray_version=25.7.25
```
#### Reality сервер yahoo.com
```
ansible-playbook xray.yml --ask-pass --ask-become-pass -e reality_site=yahoo.com
```
#### TLS сервер в домене mycompany.com
```
ansible-playbook xray.yml --ask-pass --ask-become-pass -e security=tls -e domain_name=mycompany.com
```