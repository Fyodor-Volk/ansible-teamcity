# Ansible TeamCity

Ansible-репозиторий для развертывания CI/CD-окружения на базе TeamCity.

Проект рассчитан на установку в Linux-окружениях Ubuntu/Debian и Alma/RHEL.
TeamCity Server запускается в Docker Compose, а TeamCity Agent устанавливается
на операционную систему как systemd-сервис.

## Структура проекта

- `ansible.cfg` - базовая конфигурация Ansible.
- `requirements.yml` - необходимые Ansible collections.
- `inventories/prod/hosts.yml` - пример production inventory.
- `inventories/prod/group_vars/all.yml` - общие переменные для всех хостов.
- `inventories/prod/group_vars/teamcity_server.yml` - переменные серверной роли.
- `inventories/prod/group_vars/teamcity_agents.yml` - переменные агентской роли.
- `playbooks/bootstrap.yml` - первичная подготовка серверов.
- `playbooks/teamcity_server.yml` - развертывание TeamCity Server.
- `playbooks/teamcity_agents.yml` - развертывание TeamCity Agent.
- `playbooks/site.yml` - полный запуск всех playbook.
- `roles/common` - обновление пакетов, базовые пакеты, установка Docker.
- `roles/teamcity_server` - развертывание TeamCity Server через Docker Compose.
- `roles/teamcity_agent` - установка TeamCity Agent на ОС и настройка systemd.

## Роли

### `common`

Общая роль для первичной подготовки хоста:

- обновляет пакеты;
- устанавливает базовые утилиты;
- устанавливает Docker;
- включает и запускает сервис Docker.

### `teamcity_server`

Роль для запуска серверной части TeamCity в Docker Compose.

Создает директорию `teamcity_server_compose_dir`, по умолчанию
`/home/docker/teamcity`, и размещает в ней:

- `docker-compose.yml`;
- конфигурацию `nginx`;
- директории для данных TeamCity;
- директорию для PostgreSQL.

Compose-стек содержит контейнеры:

- `teamcity-server`;
- `postgresql`;
- `nginx`.

### `teamcity_agent`

Роль для установки TeamCity Agent на операционную систему:

- устанавливает Java 21;
- создает пользователя и группу `agent-tc`;
- скачивает агент с TeamCity Server;
- распаковывает агент в `teamcity_agent_home`;
- создает `buildAgent.properties`;
- создает systemd unit `teamcity-agent.service`;
- включает и запускает сервис агента.

## Быстрый старт

Установить необходимые collections:

```bash
ansible-galaxy collection install -r requirements.yml
```

Проверить inventory:

```bash
ansible-inventory --graph
```

Запустить полный деплой:

```bash
ansible-playbook playbooks/site.yml
```

Запустить только первичную подготовку:

```bash
ansible-playbook playbooks/bootstrap.yml
```

Запустить только TeamCity Server:

```bash
ansible-playbook playbooks/teamcity_server.yml
```

Запустить только TeamCity Agents:

```bash
ansible-playbook playbooks/teamcity_agents.yml
```

## Основные переменные

### Общие

- `ansible_user` - пользователь для подключения по SSH.
- `teamcity_server_url` - URL TeamCity Server для подключения агентов.
- `teamcity_version` - версия TeamCity Docker image.
- `common_update_packages` - обновлять пакеты при bootstrap.
- `common_install_docker` - устанавливать Docker при bootstrap.

### TeamCity Server

- `teamcity_server_compose_dir` - директория для compose-файла и данных.
- `teamcity_server_http_port` - внутренний порт TeamCity Server.
- `teamcity_server_nginx_http_port` - внешний HTTP-порт nginx.
- `teamcity_postgres_db` - имя базы PostgreSQL.
- `teamcity_postgres_user` - пользователь PostgreSQL.
- `teamcity_postgres_password` - пароль PostgreSQL.

### TeamCity Agent

- `teamcity_agent_user` - пользователь для запуска агента.
- `teamcity_agent_group` - группа для запуска агента.
- `teamcity_agent_home` - директория установки агента.
- `teamcity_agent_name` - имя агента в TeamCity.
- `teamcity_agent_java_package_debian` - Java-пакет для Ubuntu/Debian.
- `teamcity_agent_java_package_redhat` - Java-пакет для Alma/RHEL.

## Подготовка перед запуском

Перед production-запуском нужно заменить примеры в inventory:

- IP-адреса хостов в `inventories/prod/hosts.yml`;
- SSH-пользователя `ansible_user`;
- `teamcity_server_url`;
- пароль `teamcity_postgres_password`;
- при необходимости путь `teamcity_server_compose_dir`.

Пароли лучше вынести в Ansible Vault.

## Проверка синтаксиса

```bash
ansible-playbook --syntax-check playbooks/site.yml
```

## Примечания

- Для Ubuntu/Debian используется пакет `openjdk-21-jdk`.
- Для Alma/RHEL используется пакет `java-21-openjdk-devel`.
- Агент скачивается с `{{ teamcity_server_url }}/update/buildAgent.zip`,
  поэтому TeamCity Server должен быть доступен агентам по сети.
