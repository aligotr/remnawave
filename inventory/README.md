# Документация Inventory

В данном разделе описана структура инвентаря Ansible для развертывания Remnawave, а также назначение основных переменных.

## Структура каталога `inventory/`

```text
inventory/
├── group_vars/             # Переменные групп хостов
│   ├── all/                # Общие переменные для всех хостов
│   │   ├── main.yml        # Основные настройки (пути, токены)
│   │   └── vault.yml       # Секреты (зашифрованы через ansible-vault)
│   ├── panel/              # Переменные для группы 'panel'
│   ├── node.yml            # Переменные для группы 'node'
│   └── subscription.yml    # Переменные для группы 'subscription'
├── host_vars/              # Индивидуальные переменные хостов
│   └── <hostname>/
│       ├── main.yml
│       └── vault.yml
└── hosts.yml               # Описание групп и хостов
```

## Группы хостов

- **`panel`**: Сервер, на котором развертывается панель управления Remnawave.
- **`node`**: Серверы-ноды (edge), на которых работает Xray/HAProxy.
- **`subscription`**: Сервер(ы), на которых развернута страница подписок (может совпадать с нодой или панелью).

## Основные переменные

### Общие (`group_vars/all/main.yml`)

| Переменная            | Описание                                                        |
| --------------------- | --------------------------------------------------------------- |
| `workdir`             | Рабочая директория для Docker-контейнеров (по умолчанию `/opt`) |
| `remnawave_panel_url` | Полный URL панели управления                                    |
| `ansible_user`        | Пользователь для SSH подключения                                |

### Панель (`group_vars/panel/main.yml`)

| Переменная                   | Описание                                      |
| ---------------------------- | --------------------------------------------- |
| `remnawave_panel_domain`     | Доменное имя панели                           |
| `remnawave_panel_port`       | Порт, на котором будет работать панель        |
| `remnawave_panel_postgres_*` | Настройки базы данных PostgreSQL              |
| `acme_domains`               | Список доменов для получения SSL сертификатов |

### Ноды (`group_vars/node.yml`)

| Переменная                     | Описание                                                    |
| ------------------------------ | ----------------------------------------------------------- |
| `remnawave_node_port`          | Порт API ноды для связи с панелью                           |
| `remnawave_sysctl_bbr_enabled` | Включение BBR оптимизации (true/false)                      |
| `system_iptables_rules_input`  | Список правил фаервола (порты 80, 443 открыты по умолчанию) |

## Примеры конфигурации

### 1. Файл `inventory/hosts.yml`

Пример разделения ролей между серверами:

```yaml
panel:
  hosts:
    panel-main: {}

node:
  hosts:
    edge-de-01: {}
    edge-nl-01: {}

subscription:
  hosts:
    edge-de-01: {} # Страница подписок будет на этой ноде
```

### 2. Настройка новой ноды (`inventory/host_vars/edge-de-01/main.yml`)

Если нужно открыть дополнительные порты на конкретной ноде:

```yaml
system_iptables_rules_input_ext:
  - { proto: "tcp", dport: "2053", comment: "Custom Xray Port" }
  - { proto: "udp", dport: "2083", comment: "Hysteria2" }
```

### 3. Секреты (`vault.yml`)

Рекомендуется хранить все чувствительные данные в зашифрованных файлах `vault.yml`. Минимальный набор секретов:

```yaml
vault_ansible_host: "1.2.3.4"
vault_remnawave_panel_api_token: "your-super-secret-token"
vault_remnawave_panel_jwt_auth_secret: "random-string"
```

## Использование

Для запуска плейбуков с использованием этого инвентаря:

```bash
# Развертывание панели
ansible-playbook playbook_remnawave_panel.yml

# Развертывание нод
ansible-playbook playbook_remnawave_nodes.yml
```
