# Remnawave Ansible

Набор Ansible плейбуков для развертывания и управления инфраструктурой Remnawave.

## Документация Inventory

Описание структуры инвентаря, групп хостов и доступных переменных находится в файле [inventory/README.md](inventory/README.md).

## Применение

### Remnawave Panel

```sh
ansible-playbook -i inventory/hosts_to_prepare.yml playbook_system_prepare.yml --limit panel
```

```sh
ansible-playbook playbook_remnawave_panel.yml --limit panel
```

## Remnawave Node

```sh
ansible-playbook -i inventory/hosts_to_prepare.yml playbook_system_prepare.yml --limit node-msk-01
```

```sh
ansible-playbook playbook_remnawave_nodes.yml
```

## Remnawave Subscription Page

Входит в состав плейбука узла:

```sh
ansible-playbook playbook_remnawave_nodes.yml --limit node
```

## Remnawave Api - Поиск доменов по имени узла

```sh
ansible-playbook playbooks/remnawave_sni_map.yml --limit node-msk-01
```

## acme.sh

```sh
ansible-playbook playbooks/remnawave_sni_map.yml playbooks/acme.yml
```

## Iptables

```sh
ansible-playbook playbooks/system_iptables.yml --limit node-msk-01
```

## Тесты

Пинг:

```sh
ansible edge-kz-01 -e "ansible_port=22" -u root --ask-pass -m ping
```

Проверка синтаксиса:

```sh
ansible-playbook playbook_remnawave_panel.yml --syntax-check
```

## Архитектура

```mermaid
flowchart TB
  User -->|remnawave-panel| PublicIP_Panel[Host1]
  User -->|remnawave-subscription-page| PublicIP_Sub[Host2]
  User -->|remnawave-node| PublicIP_Node[Host3]

  Sub3100 <-->|remnawave-api| PublicIP_Panel
  Panel3000 -->|remnawave-api| Xray3

  subgraph Host1 [Host1: panel + node]
    HAProxy1[HAProxy 80 443 ...]
    Panel3000[Panel 3000]
    Xray1[Xray]
    AcmeSh1[acme.sh]

    PublicIP_Panel --> HAProxy1
    HAProxy1 -->|sni panel.domain.com| Panel3000
    HAProxy1 -->|sni abc1.domain.com| Xray1
    HAProxy1 -->|validation| AcmeSh1
  end

  subgraph Host2 [Host2: subscription page + node]
    HAProxy2[HAProxy 80 443 ...]
    Sub3100[Subscription page 3100]
    Xray2[Xray]
    AcmeSh2[acme.sh]

    PublicIP_Sub --> HAProxy2
    HAProxy2 -->|sni subpage.domain.ru| Sub3100
    HAProxy2 -->|sni abc2.domain.com| Xray2
    HAProxy2 -->|validation| AcmeSh2
  end

  Panel3000 -->|remnawave-api| Xray2

  subgraph Host3 [Host3: node]
    HAProxy3[HAProxy 80 443 ...]
    Xray3[Xray]
    AcmeSh3[acme.sh]

    PublicIP_Node --> HAProxy3
    HAProxy3 -->|sni abc3.domain.com| Xray3
    HAProxy3 -->|validation| AcmeSh3
  end
```

---

_"remnawave_sni_map" и "remnawave_inbounds_cache" взяты из проекта [vff-remnawave-auto](https://github.com/ryabkov82/vff-remnawave-auto)._
