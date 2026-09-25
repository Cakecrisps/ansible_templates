# ansible_templates

Набор Ansible-плейбуков и ролей для базового хардненинга серверов Debian/Ubuntu и развёртывания nginx.

## Структура

```
ansible_templates/
├── inventory.ini
├── playbook.yml
└── roles/
    ├── hardening/
    │   ├── defaults/main.yml
    │   ├── handlers/main.yml
    │   └── tasks/
    │       ├── main.yml
    │       ├── ssh.yml
    │       ├── user.yml
    │       └── auditd.yml
    └── nginx/
        ├── defaults/main.yml
        ├── handlers/main.yml
        ├── tasks/main.yml
        └── templates/
```

## Плейбук

`playbook.yml` содержит два play:

| Play | Хосты | Роль |
|---|---|---|
| Apply server hardening | `servers` | `hardening` |
| NginxConf | `webservers` | `nginx` |

## Роль `hardening`

Приводит сервер к базовому защищённому состоянию:

- **SSH** (`tasks/ssh.yml`) — установка `openssh-server`, деплой кастомного `sshd_config` из шаблона с валидацией конфига перед применением (`sshd -t`), запуск и автозагрузка службы.
- **Административный пользователь** (`tasks/user.yml`) — создание пользователя (`ssh_admin_user`), добавление в группу `sudo`, создание `~/.ssh` с правами `0700` и установка публичного SSH-ключа через `authorized_key`.
- **Аудит журналов** (`tasks/auditd.yml`) — установка `auditd` и `audispd-plugins`, деплой правил аудита (`base.rules`), запуск и автозагрузка `auditd`.

### Переменные по умолчанию (`roles/hardening/defaults/main.yml`)

| Переменная | Значение по умолчанию | Описание |
|---|---|---|
| `ssh_port` | `22` | Порт SSH |
| `ssh_admin_user` | `k4ips` | Имя создаваемого администратора |
| `ssh_public_key` | `ssh-ed25519 ...` | Публичный ключ для входа |
| `ssh_allowed_users` | `[k4ips, admin]` | Пользователи, которым разрешён вход по SSH |

### Хендлеры

- `Restart SSH`
- `Restart auditd`
- `Reload audit rules` (`augenrules --load`)

## Роль `nginx`

- Устанавливает пакет `nginx`.
- Проверяет/запускает и включает автозагрузку сервиса.
- Деплоит `index.html` и конфиг `sites-available/default` из шаблонов Jinja2 (`index.html.j2`, `default.j2`).
- При изменении шаблонов вызывает хендлер `Restart Nginx`.

### Переменные по умолчанию (`roles/nginx/defaults/main.yml`)

| Переменная | Значение по умолчанию | Описание |
|---|---|---|
| `nginx_port` | `80` | Порт nginx |
| `main_page_word` | `ANSIBLE MANAGE THIS` | Текст на главной странице |

## Inventory

Пример `inventory.ini` описывает группы `servers` и `webservers` с подключением по SSH-ключу и `become: true`.

```ini
[servers]
debian-base ansible_host=... ansible_user=admin ansible_ssh_private_key_file=./adminkey ...

[webservers]
debian-base2 ansible_host=... ansible_user=admin ansible_ssh_private_key_file=./adminkey ...
```

## Использование

1. Отредактируйте `inventory.ini` под свои хосты и путь к приватному ключу.
2. При необходимости переопределите переменные ролей (`ssh_admin_user`, `ssh_public_key` и т. д.) в `group_vars` / `host_vars` или через `-e`.
3. Запустите плейбук:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

## Требования

- Ansible ≥ 2.10
- Целевые хосты — Debian/Ubuntu
- SSH-доступ с правами на `become` (sudo)

## Лицензия

Не указана.
