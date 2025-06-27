
# Ansible Playbooks

Este projeto tem como objetivo automatizar a atualização do sistema e a instalação de serviços como Docker, Apache, NGINX, NTP, Zabbix Agent e HAProxy em servidores Ubuntu 24.04.2 usando Ansible.

---

## Estrutura do Projeto

```
ansible-docker/
├── inventories/
│   └── production/
│       ├── hosts.ini
│       └── group_vars/
│           └── all.yml
├── playbooks/
│   ├── update-system.yml
│   ├── install-docker.yml
│   ├── install-nginx.yml
│   ├── install-apache.yml
│   ├── install-ntp.yml
│   ├── install-zabbix-agent.yml
│   ├── install-haproxy.yml
│   └── multi-install.yml
├── roles/
│   ├── common/
│   │   └── tasks/main.yml
│   ├── docker/
│   │   └── tasks/main.yml
│   ├── nginx/
│   │   └── tasks/main.yml
│   ├── apache/
│   │   └── tasks/main.yml
│   ├── ntp/
│   │   └── tasks/main.yml
│   ├── zabbix/
│   │   └── tasks/main.yml
│   └── haproxy/
│       └── tasks/main.yml
└── README.md
```

---

## Pré-requisitos

- Servidores Ubuntu 24.04.2 acessíveis via SSH
- Um usuário com permissões sudo (ex: `devops`)
- Chave SSH configurada
- Máquina de controle com Ansible instalado

```bash
sudo apt update && sudo apt install -y ansible sshpass
```

---

## Inventário (`inventories/production/hosts.ini`)

```ini
[ubuntu_servers]
192.168.1.10
192.168.1.11
192.168.1.12

[ubuntu_servers:vars]
ansible_user=devops
ansible_ssh_private_key_file=~/.ssh/id_ed25519
ansible_python_interpreter=/usr/bin/python3
```

---

## Variáveis Dinâmicas (`group_vars/all.yml`)

Exemplo de variáveis configuráveis para instalação, ajute conforme a necessidade:

```yaml
# Docker
docker_compose_version: "2.26.1"
docker_compose_path: "/usr/local/bin/docker-compose"

# NGINX
nginx_default_port: 80

# Apache
apache_default_port: 80

# NTP
ntp_server: "pool.ntp.org"

# Zabbix
zabbix_server: "192.168.1.100"
zabbix_agentd_conf: "/etc/zabbix/zabbix_agentd.conf"

# HAProxy
haproxy_cfg_path: "/etc/haproxy/haproxy.cfg"
```

As variáveis podem ser referenciadas nos templates ou diretamente nos `tasks/main.yml` de cada role.

---

## Executando Playbooks

Executando os playbooks de forma separada:

```bash
# Atualizar o sistema
ansible-playbook -i inventories/production/hosts.ini playbooks/update-system.yml

# Instalar Docker
ansible-playbook -i inventories/production/hosts.ini playbooks/install-docker.yml

# Instalar Apache
ansible-playbook -i inventories/production/hosts.ini playbooks/install-apache.yml

# Instalar NGINX
ansible-playbook -i inventories/production/hosts.ini playbooks/install-nginx.yml

# Instalar NTP
ansible-playbook -i inventories/production/hosts.ini playbooks/install-ntp.yml

# Instalar Zabbix Agent
ansible-playbook -i inventories/production/hosts.ini playbooks/install-zabbix-agent.yml

# Instalar HAProxy
ansible-playbook -i inventories/production/hosts.ini playbooks/install-haproxy.yml
```

---

## Executando por Tags

Combinação de instalação de serviços usando o playbook `combo.yml` com tags:

```yaml
---
- name: Multi Install Services
  hosts: ubuntu_servers
  become: true
  roles:
    - { role: common, tags: ["update"] }
    - { role: nginx, tags: ["nginx"] }
    - { role: ntp, tags: ["ntp"] }
    - { role: docker, tags: ["docker"] }
    - { role: apache, tags: ["apache"] }
    - { role: zabbix, tags: ["zabbix"] }
    - { role: haproxy, tags: ["haproxy"] }
```

Exemplo:

```bash
ansible-playbook -i inventories/production/hosts.ini playbooks/combo.yml --tags "update,nginx,ntp"
```
