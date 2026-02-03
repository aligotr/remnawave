# Remnawave Ansible

Набор Ansible плейбуков для развертывания и управления инфраструктурой Remnawave.

## Документация Inventory

Подробное описание структуры инвентаря, групп хостов и доступных переменных находится в файле [INVENTORY.md](inventory/README.md).

## Применение

### Remnawave Panel

`ansible-playbook -i inventory/hosts_to_prepare.yml playbook_system_prepare.yml --limit panel`
`ansible-playbook playbook_remnawave_panel.yml --limit panel`

## Remnawave Node

`ansible-playbook -i inventory/hosts_to_prepare.yml playbook_system_prepare.yml --limit node-msk-01`
`ansible-playbook playbook_remnawave_nodes.yml`

## Remnawave Subscription Page

Входит в состав плейбука узла:
`ansible-playbook playbook_remnawave_nodes.yml --limit node`

## acme.sh

`ansible-playbook playbooks/remnawave_sni_map.yml playbooks/acme.yml`

## Remnawave Api - Поиск доменов по имени узла

`ansible-playbook playbooks/remnawave_sni_map.yml --limit node-msk-01`

## Iptables

`ansible-playbook playbooks/system_iptables.yml --limit node-msk-01`

## Тесты

Пинг: `ansible edge-kz-01 -e "ansible_port=22" -u root --ask-pass -m ping`
Проверка синтаксиса: `ansible-playbook playbook_remnawave_panel.yml --syntax-check`
