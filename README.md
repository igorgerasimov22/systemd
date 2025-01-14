# systemd

Репозиторий содержит примеры конфигураций и инструкций для работы с несколькими экземплярами (`instances`) сервиса Nginx с использованием systemd. 

Этот проект демонстрирует, как можно запускать несколько экземпляров Nginx с разными конфигурациями на одном сервере, управляя ими через systemd.

## Основные возможности
- **Создание нескольких экземпляров Nginx:**
  - Разделение конфигурационных файлов для каждого экземпляра.
  - Индивидуальные PID-файлы, лог-файлы и параметры запуска.
- **Автоматизация управления экземплярами:**
  - Инструкции по созданию unit-файлов для `systemd`.
  - Автоматическая перезагрузка конфигурации при изменении файлов.
- **Использование Ansible для автоматизации:**
  - Playbook для управления `systemd` и перезагрузки демонов.

## Структура проекта

```plaintext
├── ansible/
│   ├── inventory.ini           # Файл инвентаря для Ansible.
│   ├── nginx-playbook.yml      # Playbook для настройки и управления Nginx.
│   └── templates/
│       ├── nginx@.service.j2   # Шаблон unit-файла для экземпляров Nginx.
│       └── nginx.conf.j2       # Шаблон конфигурации Nginx.
├── systemd/
│   ├── nginx@.service          # Пример unit-файла для Nginx.
│   └── nginx-first.conf        # Пример конфигурации для первого экземпляра.
└── README.md                   # Файл описания проекта.
```

## Установка и использование
### 1. Клонирование репозитория
```bash
git clone https://github.com/igorgerasimov22/systemd.git
cd systemd
```
### 2. Настройка экземпляров Nginx
#### Шаги
1. Скопируйте ```nginx@.service``` в директорию systemd:
```bash
sudo cp systemd/nginx@.service /etc/systemd/system/
```
2. Создайте конфигурацию для каждого экземпляра в ```/etc/nginx/```:
```bash
sudo cp systemd/nginx-first.conf /etc/nginx/nginx-first.conf
```
3. Запустите первый экземпляр:
```bash
sudo systemctl start nginx@first
```

### 3. Автоматизация с Ansible
1. Отредактируйте ```inventory.ini```, указав свои серверы.
2. Запустите playbook:
```bash
ansible-playbook -i ansible/inventory.ini ansible/nginx-playbook.yml
```

## Требования
* Операционная система: Ubuntu 20.04 или новее.
* Установленные пакеты:
    * ```nginx```
    * ```systemd```
    * ```ansible``` (если используется автоматизация).

## Пример ```nginx@.service```
Пример unit-файла для запуска экземпляров Nginx:
```ini
[Unit]
Description=A high performance web server and a reverse proxy server - Instance %i
After=network.target

[Service]
Type=forking
PIDFile=/run/nginx-%i.pid
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-%i.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

[Install]
WantedBy=multi-user.target
```