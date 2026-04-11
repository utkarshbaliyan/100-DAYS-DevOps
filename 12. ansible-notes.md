# 📘 Ansible — Comprehensive DevOps Interview Notes

> **Purpose:** Complete revision guide from basics to advanced concepts.  
> **Format:** Structured for GitHub upload & quick interview revision.

---

## Table of Contents

1. [What is Ansible?](#1-what-is-ansible)
2. [Architecture & Components](#2-architecture--components)
3. [Installation & Setup](#3-installation--setup)
4. [Inventory](#4-inventory)
5. [Ad-Hoc Commands](#5-ad-hoc-commands)
6. [Playbooks](#6-playbooks)
7. [Variables](#7-variables)
8. [Facts](#8-facts)
9. [Conditionals](#9-conditionals)
10. [Loops](#10-loops)
11. [Handlers](#11-handlers)
12. [Templates (Jinja2)](#12-templates-jinja2)
13. [Roles](#13-roles)
14. [Ansible Vault](#14-ansible-vault)
15. [Tags](#15-tags)
16. [Error Handling](#16-error-handling)
17. [Modules (Key Ones)](#17-modules-key-ones)
18. [Ansible Galaxy](#18-ansible-galaxy)
19. [Dynamic Inventory](#19-dynamic-inventory)
20. [Ansible Tower / AWX](#20-ansible-tower--awx)
21. [Best Practices](#21-best-practices)
22. [Interview Q&A Cheatsheet](#22-interview-qa-cheatsheet)

---

## 1. What is Ansible?

Ansible is an **open-source IT automation tool** developed by Red Hat. It automates:
- Configuration management
- Application deployment
- Orchestration
- Provisioning

### Key Characteristics

| Feature | Description |
|---|---|
| **Agentless** | No agent needed on managed nodes; uses SSH (Linux) or WinRM (Windows) |
| **Idempotent** | Running the same playbook multiple times produces the same result |
| **Declarative** | You describe the desired state, not the steps to get there |
| **YAML-based** | Human-readable configuration language |
| **Push-based** | Control node pushes configuration to managed nodes |

### Ansible vs. Other Tools

| Tool | Agent | Language | Style |
|---|---|---|---|
| Ansible | Agentless | YAML | Declarative |
| Chef | Agent | Ruby DSL | Imperative |
| Puppet | Agent | Puppet DSL | Declarative |
| SaltStack | Agent (optional) | YAML | Declarative |

---

## 2. Architecture & Components

```
┌─────────────────────────────────────────────┐
│              CONTROL NODE                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │Inventory │  │Playbooks │  │  Modules │  │
│  └──────────┘  └──────────┘  └──────────┘  │
└────────────────────┬────────────────────────┘
                     │ SSH / WinRM
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   [Node 1]      [Node 2]     [Node 3]
  (Managed)     (Managed)    (Managed)
```

### Core Components

- **Control Node** — Machine where Ansible is installed and commands run from.
- **Managed Nodes** — Servers Ansible manages (no Ansible installation required).
- **Inventory** — List of managed nodes (hosts/groups).
- **Playbook** — YAML file containing a set of tasks to be executed.
- **Task** — A single unit of action (e.g., install a package).
- **Module** — Reusable unit of code that performs a specific task (e.g., `apt`, `copy`, `service`).
- **Plugin** — Extends Ansible's functionality (connection plugins, callback plugins, etc.).
- **Role** — Reusable, structured collection of tasks, variables, handlers, and templates.
- **Ansible Galaxy** — Community hub for sharing roles and collections.

---

## 3. Installation & Setup

### Install Ansible (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
```

### Install via pip

```bash
pip install ansible
```

### Verify Installation

```bash
ansible --version
```

### Configure SSH Key-Based Authentication

```bash
# Generate SSH key on control node
ssh-keygen -t rsa -b 4096

# Copy public key to managed nodes
ssh-copy-id user@managed-node-ip

# Test connection
ansible all -m ping -i inventory.ini
```

### Ansible Configuration File (`ansible.cfg`)

Ansible looks for config in this order:
1. `ANSIBLE_CONFIG` environment variable
2. `./ansible.cfg` (current directory)
3. `~/.ansible.cfg`
4. `/etc/ansible/ansible.cfg`

```ini
[defaults]
inventory       = ./inventory.ini
remote_user     = ubuntu
private_key_file = ~/.ssh/id_rsa
host_key_checking = False
retry_files_enabled = False
forks           = 10

[privilege_escalation]
become          = True
become_method   = sudo
become_user     = root
```

---

## 4. Inventory

Inventory defines the hosts and groups Ansible manages.

### Static Inventory (INI format) — `inventory.ini`

```ini
# Single host
192.168.1.10

# Group of hosts
[webservers]
web1.example.com
web2.example.com ansible_user=ubuntu

[dbservers]
db1.example.com ansible_port=2222
db2.example.com

# Group of groups
[production:children]
webservers
dbservers

# Variables for a group
[webservers:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python3

# Variables for a host (inline)
web1.example.com ansible_host=192.168.1.5 ansible_user=admin
```

### Static Inventory (YAML format) — `inventory.yml`

```yaml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.10
          ansible_user: ubuntu
        web2:
          ansible_host: 192.168.1.11
    dbservers:
      hosts:
        db1:
          ansible_host: 192.168.1.20
          ansible_port: 3306
  vars:
    ansible_python_interpreter: /usr/bin/python3
```

### Host & Group Variables (Directory structure)

```
inventory/
├── hosts.ini
├── host_vars/
│   ├── web1.yml       # Variables specific to web1
│   └── db1.yml
└── group_vars/
    ├── all.yml        # Variables for ALL hosts
    ├── webservers.yml # Variables for webservers group
    └── dbservers.yml
```

### Common Inventory Variables

| Variable | Description |
|---|---|
| `ansible_host` | IP or hostname to connect to |
| `ansible_port` | SSH port (default: 22) |
| `ansible_user` | SSH user |
| `ansible_password` | SSH password (avoid; use keys) |
| `ansible_ssh_private_key_file` | Path to private key |
| `ansible_become` | Enable privilege escalation |
| `ansible_become_user` | User to escalate to |
| `ansible_python_interpreter` | Python path on managed node |
| `ansible_connection` | Connection type (`ssh`, `local`, `winrm`) |

---

## 5. Ad-Hoc Commands

Ad-hoc commands are one-liners for quick tasks — no playbook needed.

### Syntax

```bash
ansible <host-pattern> -m <module> -a "<arguments>" [options]
```

### Common Examples

```bash
# Ping all hosts
ansible all -m ping

# Run a shell command
ansible webservers -m shell -a "uptime"

# Copy a file
ansible all -m copy -a "src=/tmp/test.txt dest=/tmp/test.txt"

# Install a package
ansible webservers -m apt -a "name=nginx state=present" --become

# Restart a service
ansible webservers -m service -a "name=nginx state=restarted" --become

# Gather facts from a host
ansible web1 -m setup

# Check disk space
ansible all -m shell -a "df -h"

# Create a user
ansible all -m user -a "name=devops state=present" --become

# Fetch a file from remote
ansible web1 -m fetch -a "src=/etc/hosts dest=/tmp/"

# Limit to specific host
ansible webservers -m ping --limit web1

# Run as a specific user
ansible all -m ping -u ec2-user --private-key=~/.ssh/mykey.pem
```

### Common Flags

| Flag | Description |
|---|---|
| `-i` | Specify inventory file |
| `-m` | Module name |
| `-a` | Module arguments |
| `-b` / `--become` | Privilege escalation (sudo) |
| `--become-user` | User to become (default: root) |
| `-u` | Remote user |
| `-k` | Prompt for SSH password |
| `-K` | Prompt for sudo password |
| `-v / -vv / -vvv` | Verbosity level |
| `--check` | Dry run (no changes) |
| `--diff` | Show file differences |
| `-f` | Number of forks (parallel hosts) |
| `--limit` | Limit to a subset of hosts |

---

## 6. Playbooks

A **playbook** is a YAML file containing one or more **plays**. Each play maps a group of hosts to a set of **tasks**.

### Basic Playbook Structure

```yaml
---
- name: Setup Web Server           # Play name
  hosts: webservers                # Target hosts/groups
  become: true                     # Enable sudo
  become_user: root
  gather_facts: true               # Collect system facts (default: true)
  vars:
    http_port: 80
    package_name: nginx

  tasks:
    - name: Install nginx
      apt:
        name: "{{ package_name }}"
        state: present
        update_cache: yes

    - name: Start and enable nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Copy index.html
      copy:
        src: files/index.html
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: '0644'

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

### Running Playbooks

```bash
# Basic run
ansible-playbook playbook.yml

# With custom inventory
ansible-playbook -i inventory.ini playbook.yml

# Dry run
ansible-playbook playbook.yml --check

# Show diffs
ansible-playbook playbook.yml --diff

# Limit to specific hosts
ansible-playbook playbook.yml --limit webservers

# Extra variables
ansible-playbook playbook.yml -e "env=prod version=1.2"

# Verbose
ansible-playbook playbook.yml -vvv

# Start at a specific task
ansible-playbook playbook.yml --start-at-task="Install nginx"

# Run specific tags
ansible-playbook playbook.yml --tags "install,configure"

# Skip tags
ansible-playbook playbook.yml --skip-tags "testing"

# Syntax check
ansible-playbook playbook.yml --syntax-check

# List hosts
ansible-playbook playbook.yml --list-hosts

# List tasks
ansible-playbook playbook.yml --list-tasks
```

### Multi-play Playbook

```yaml
---
- name: Configure Web Servers
  hosts: webservers
  become: true
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

- name: Configure DB Servers
  hosts: dbservers
  become: true
  tasks:
    - name: Install MySQL
      apt:
        name: mysql-server
        state: present
```

---

## 7. Variables

### Defining Variables

#### In a Playbook

```yaml
vars:
  app_name: myapp
  app_version: "2.1"
  app_port: 8080
  db_config:
    host: localhost
    port: 3306
    name: mydb
  allowed_ips:
    - 10.0.0.1
    - 10.0.0.2
```

#### In a Separate Vars File

```yaml
# vars/main.yml
app_name: myapp
app_port: 8080
```

```yaml
# In playbook
vars_files:
  - vars/main.yml
  - vars/secrets.yml
```

#### Prompted from User

```yaml
vars_prompt:
  - name: username
    prompt: "Enter the username"
    private: no
  - name: password
    prompt: "Enter the password"
    private: yes    # hides input
```

#### Via Command Line

```bash
ansible-playbook playbook.yml -e "env=production version=2.0"
ansible-playbook playbook.yml -e "@vars/extra.yml"
```

### Using Variables

```yaml
tasks:
  - name: Create directory
    file:
      path: "/opt/{{ app_name }}"
      state: directory

  - name: Print variable
    debug:
      msg: "App: {{ app_name }}, Version: {{ app_version }}"

  - name: Access dict variable
    debug:
      msg: "DB Host: {{ db_config.host }}"   # or db_config['host']

  - name: Access list variable
    debug:
      msg: "First IP: {{ allowed_ips[0] }}"
```

### Variable Precedence (lowest to highest)

```
1.  Role defaults          (roles/x/defaults/main.yml)
2.  Inventory group_vars/all
3.  Inventory group_vars/*
4.  Playbook group_vars/all
5.  Playbook group_vars/*
6.  Inventory host_vars/*
7.  Playbook host_vars/*
8.  Host facts / cached set_facts
9.  Play vars
10. Play vars_prompt
11. Play vars_files
12. Role vars               (roles/x/vars/main.yml)
13. Block vars
14. Task vars
15. include_vars
16. set_facts / registered vars
17. Role (and include_role) params
18. include_params
19. Extra vars (-e)          ← HIGHEST PRIORITY
```

### Registering Variables

```yaml
tasks:
  - name: Check if file exists
    stat:
      path: /etc/nginx/nginx.conf
    register: nginx_conf

  - name: Print result
    debug:
      msg: "File exists: {{ nginx_conf.stat.exists }}"

  - name: Run command and capture output
    shell: cat /etc/os-release
    register: os_info

  - name: Show output
    debug:
      var: os_info.stdout_lines
```

### set_fact

```yaml
- name: Set a computed variable
  set_fact:
    full_app_path: "/opt/{{ app_name }}/{{ app_version }}"
    is_production: "{{ env == 'prod' }}"
```

---

## 8. Facts

**Facts** are system information automatically gathered from managed nodes using the `setup` module.

### Gather Facts

```bash
# View all facts for a host
ansible web1 -m setup

# Filter facts
ansible web1 -m setup -a "filter=ansible_os_family"
```

### Commonly Used Facts

```yaml
ansible_hostname           # Hostname
ansible_fqdn               # Fully Qualified Domain Name
ansible_os_family          # OS family: Debian, RedHat, etc.
ansible_distribution       # Ubuntu, CentOS, etc.
ansible_distribution_version  # 20.04, 8.5, etc.
ansible_architecture       # x86_64, arm64
ansible_memtotal_mb        # Total RAM in MB
ansible_processor_cores    # CPU cores
ansible_default_ipv4.address  # Primary IP
ansible_interfaces         # Network interfaces
ansible_env                # Environment variables
ansible_date_time          # Date and time info
ansible_user_dir           # Home directory
ansible_python_version     # Python version
```

### Using Facts in Tasks

```yaml
- name: Install Apache on Debian/Ubuntu
  apt:
    name: apache2
    state: present
  when: ansible_os_family == "Debian"

- name: Install Apache on RedHat/CentOS
  yum:
    name: httpd
    state: present
  when: ansible_os_family == "RedHat"
```

### Custom Facts

Create a file on managed node: `/etc/ansible/facts.d/custom.fact`

```ini
[app]
version=2.1
environment=production
```

Access in playbook:

```yaml
- debug:
    msg: "{{ ansible_local.custom.app.version }}"
```

### Disabling Fact Gathering

```yaml
- hosts: all
  gather_facts: false   # Speeds up playbook when facts aren't needed
```

---

## 9. Conditionals

### `when` Statement

```yaml
tasks:
  - name: Install on Debian systems only
    apt:
      name: vim
      state: present
    when: ansible_os_family == "Debian"

  - name: Multiple conditions (AND)
    apt:
      name: nginx
      state: present
    when:
      - ansible_os_family == "Debian"
      - ansible_distribution_version == "20.04"

  - name: OR condition
    debug:
      msg: "It's Debian or RedHat"
    when: ansible_os_family == "Debian" or ansible_os_family == "RedHat"

  - name: Check if variable is defined
    debug:
      msg: "Variable is set"
    when: my_var is defined

  - name: Check if variable is not defined
    debug:
      msg: "Variable is not set"
    when: my_var is undefined

  - name: Check registered variable
    command: systemctl is-active nginx
    register: nginx_status
    ignore_errors: true

  - name: Print if nginx is running
    debug:
      msg: "Nginx is running"
    when: nginx_status.rc == 0

  - name: Check string contains
    debug:
      msg: "Ubuntu system"
    when: "'Ubuntu' in ansible_distribution"

  - name: Check list membership
    debug:
      msg: "This host is a web server"
    when: inventory_hostname in groups['webservers']
```

### Jinja2 Tests (for `when`)

```yaml
when: result is success
when: result is failed
when: result is changed
when: result is skipped
when: my_var is defined
when: my_var is none
when: my_var | bool
when: my_list | length > 0
```

---

## 10. Loops

### `loop` (Modern approach)

```yaml
- name: Install multiple packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - git
    - curl
    - vim

- name: Create multiple users
  user:
    name: "{{ item.name }}"
    state: present
    groups: "{{ item.groups }}"
  loop:
    - { name: alice, groups: sudo }
    - { name: bob, groups: developers }
    - { name: charlie, groups: developers }

- name: Loop with index
  debug:
    msg: "Item {{ ansible_loop.index }}: {{ item }}"
  loop:
    - apple
    - banana
    - cherry
  loop_control:
    index_var: my_index
    label: "{{ item }}"   # Simplifies output display
```

### `with_*` Directives (Legacy but still used)

```yaml
# with_items (equivalent to loop for flat lists)
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - nginx
    - git

# with_dict
- name: Configure settings
  debug:
    msg: "{{ item.key }} = {{ item.value }}"
  with_dict:
    key1: value1
    key2: value2

# with_fileglob — loop over files matching a pattern
- name: Copy config files
  copy:
    src: "{{ item }}"
    dest: /etc/myapp/
  with_fileglob:
    - "files/*.conf"

# with_sequence — generate a sequence
- name: Create numbered directories
  file:
    path: "/data/dir{{ item }}"
    state: directory
  with_sequence: start=1 end=5

# with_nested — nested loops
- name: Nested loop
  debug:
    msg: "{{ item[0] }} - {{ item[1] }}"
  with_nested:
    - [a, b]
    - [1, 2]
```

### Loop Control Options

```yaml
loop_control:
  label: "{{ item.name }}"    # Custom label in output
  index_var: idx              # Variable to hold index
  loop_var: outer_item        # Rename loop variable (useful in nested includes)
  pause: 2                    # Pause N seconds between iterations
```

---

## 11. Handlers

Handlers are tasks that run **only when notified** by another task. They run **once** at the end of the play, regardless of how many times they're notified.

### Basic Handler

```yaml
tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present
    notify: restart nginx         # Notify handler by name

  - name: Update nginx config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: restart nginx         # Same handler notified again (runs only once)

handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

### Multiple Handlers

```yaml
tasks:
  - name: Update config
    copy:
      src: app.conf
      dest: /etc/app/app.conf
    notify:
      - validate config
      - restart app

handlers:
  - name: validate config
    command: /usr/bin/app --validate-config

  - name: restart app
    service:
      name: myapp
      state: restarted
```

### Force Handlers to Run

```yaml
# Run handlers mid-play (not at end)
- meta: flush_handlers
```

### Handlers in Roles

```yaml
# roles/nginx/handlers/main.yml
- name: restart nginx
  service:
    name: nginx
    state: restarted

- name: reload nginx
  service:
    name: nginx
    state: reloaded
```

---

## 12. Templates (Jinja2)

Templates allow dynamic file generation using **Jinja2** templating engine. Template files use `.j2` extension.

### Using the `template` Module

```yaml
- name: Deploy nginx config
  template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
  notify: reload nginx
```

### Jinja2 Template Syntax

#### Variables

```jinja2
server {
    listen {{ http_port }};
    server_name {{ ansible_hostname }};
    root {{ web_root }};
}
```

#### Conditionals

```jinja2
{% if enable_ssl %}
    listen 443 ssl;
    ssl_certificate /etc/ssl/cert.pem;
{% else %}
    listen 80;
{% endif %}
```

#### Loops

```jinja2
{% for server in backend_servers %}
    server {{ server.host }}:{{ server.port }};
{% endfor %}
```

#### Filters

```jinja2
{{ my_var | upper }}
{{ my_var | lower }}
{{ my_var | default('fallback') }}
{{ my_list | join(', ') }}
{{ my_var | int }}
{{ my_var | string }}
{{ my_var | bool }}
{{ my_var | length }}
{{ my_var | replace('old', 'new') }}
{{ my_var | regex_replace('^foo', 'bar') }}
{{ my_dict | to_json }}
{{ my_dict | to_yaml }}
{{ my_var | b64encode }}
{{ my_var | b64decode }}
{{ my_var | hash('sha256') }}
{{ my_var | password_hash('sha512') }}
```

#### Example Template — `nginx.conf.j2`

```jinja2
# Managed by Ansible — do not edit manually
user www-data;
worker_processes {{ nginx_worker_processes | default('auto') }};

events {
    worker_connections {{ nginx_worker_connections | default(1024) }};
}

http {
    upstream backend {
{% for host in backend_hosts %}
        server {{ host }}:{{ backend_port }};
{% endfor %}
    }

    server {
        listen {{ http_port }};
        server_name {{ server_name }};

{% if enable_gzip | default(false) %}
        gzip on;
        gzip_types text/plain application/json;
{% endif %}

        location / {
            proxy_pass http://backend;
        }
    }
}
```

---

## 13. Roles

Roles are a way to organize playbooks into **reusable, self-contained units**.

### Role Directory Structure

```
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml          # Main task list
    ├── handlers/
    │   └── main.yml          # Handlers
    ├── templates/
    │   └── nginx.conf.j2     # Jinja2 templates
    ├── files/
    │   └── index.html        # Static files to copy
    ├── vars/
    │   └── main.yml          # Role variables (high precedence)
    ├── defaults/
    │   └── main.yml          # Default variables (low precedence)
    ├── meta/
    │   └── main.yml          # Role metadata & dependencies
    └── README.md
```

### Creating a Role

```bash
# Generate role scaffold
ansible-galaxy role init nginx

# Or manually create the structure
mkdir -p roles/nginx/{tasks,handlers,templates,files,vars,defaults,meta}
```

### Role Task File — `roles/nginx/tasks/main.yml`

```yaml
---
- name: Install nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Deploy configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx

- name: Ensure nginx is running
  service:
    name: nginx
    state: started
    enabled: yes
```

### Role Defaults — `roles/nginx/defaults/main.yml`

```yaml
---
http_port: 80
nginx_worker_processes: auto
enable_ssl: false
```

### Role Vars — `roles/nginx/vars/main.yml`

```yaml
---
nginx_config_path: /etc/nginx/nginx.conf
nginx_log_dir: /var/log/nginx
```

### Role Meta — `roles/nginx/meta/main.yml`

```yaml
---
galaxy_info:
  author: your_name
  description: Nginx web server role
  license: MIT
  min_ansible_version: "2.9"
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy

dependencies:
  - role: common
  - role: firewall
    vars:
      firewall_port: 80
```

### Using Roles in a Playbook

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true
  roles:
    - common
    - nginx
    - { role: nodejs, vars: { node_version: "18" } }

# Or using include_role for more control
- name: Configure servers
  hosts: all
  tasks:
    - name: Include nginx role
      include_role:
        name: nginx
      vars:
        http_port: 8080
      when: install_nginx | default(false)
```

---

## 14. Ansible Vault

Ansible Vault encrypts sensitive data (passwords, keys, secrets).

### Encrypting Files

```bash
# Create a new encrypted file
ansible-vault create secrets.yml

# Encrypt an existing file
ansible-vault encrypt vars/secrets.yml

# Edit an encrypted file
ansible-vault edit vars/secrets.yml

# View without decrypting to disk
ansible-vault view vars/secrets.yml

# Decrypt a file
ansible-vault decrypt vars/secrets.yml

# Re-key (change password)
ansible-vault rekey vars/secrets.yml

# Encrypt a single string (embed in YAML)
ansible-vault encrypt_string 'my_secret_password' --name 'db_password'
```

### Encrypted String in Playbook

```yaml
vars:
  db_password: !vault |
    $ANSIBLE_VAULT;1.1;AES256
    3061323932346438383738373...
    6163666164336161363939...
```

### Running Playbooks with Vault

```bash
# Prompt for vault password
ansible-playbook playbook.yml --ask-vault-pass

# Use a password file
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass

# Use environment variable
export ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass
ansible-playbook playbook.yml

# Multiple vault IDs
ansible-vault create --vault-id dev@prompt dev_secrets.yml
ansible-playbook playbook.yml --vault-id dev@~/.vault_dev
```

### Best Practices for Vault

```
group_vars/
├── all/
│   ├── vars.yml       # Plain variables
│   └── vault.yml      # Encrypted secrets (ansible-vault encrypt)
```

```yaml
# vars.yml — reference vault variables
db_password: "{{ vault_db_password }}"

# vault.yml — encrypted file containing
vault_db_password: "super_secret_password"
```

---

## 15. Tags

Tags allow you to run or skip specific tasks in a playbook.

### Assigning Tags

```yaml
tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present
    tags:
      - install
      - nginx

  - name: Configure nginx
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    tags:
      - configure
      - nginx

  - name: Start nginx
    service:
      name: nginx
      state: started
    tags:
      - start
      - nginx

  - name: Always run this task
    debug:
      msg: "This always runs"
    tags:
      - always    # Special tag: always runs unless --skip-tags always
```

### Tags on Blocks and Plays

```yaml
- name: Web setup play
  hosts: webservers
  tags: websetup       # Tag the entire play

  tasks:
    - block:
        - name: Task 1
          debug: msg="Task 1"
        - name: Task 2
          debug: msg="Task 2"
      tags: myblock    # Tag the entire block
```

### Running with Tags

```bash
# Run only tasks with specific tag
ansible-playbook playbook.yml --tags "install"

# Run multiple tags
ansible-playbook playbook.yml --tags "install,configure"

# Skip specific tags
ansible-playbook playbook.yml --skip-tags "testing"

# List all tags in a playbook
ansible-playbook playbook.yml --list-tags

# Run everything except "never" tagged tasks
ansible-playbook playbook.yml   # "never" tagged tasks are skipped by default
```

### Special Tags

| Tag | Behavior |
|---|---|
| `always` | Task always runs, even if other tags are specified |
| `never` | Task never runs unless explicitly called with `--tags never` |
| `tagged` | Run all tagged tasks |
| `untagged` | Run all untagged tasks |
| `all` | Run all tasks (default) |

---

## 16. Error Handling

### `ignore_errors`

```yaml
- name: Try to stop a service (ignore if not found)
  service:
    name: oldapp
    state: stopped
  ignore_errors: yes
```

### `failed_when`

```yaml
- name: Run a command
  shell: /usr/bin/check-status
  register: result
  failed_when: "'ERROR' in result.stdout"

- name: Another example
  command: echo "success"
  register: cmd_result
  failed_when:
    - cmd_result.rc != 0
    - "'success' not in cmd_result.stdout"
```

### `changed_when`

```yaml
- name: Run idempotent script
  shell: /usr/bin/deploy.sh
  register: deploy_result
  changed_when: "'already up to date' not in deploy_result.stdout"
```

### `block`, `rescue`, `always`

```yaml
tasks:
  - block:
      - name: Attempt risky task
        command: /usr/bin/risky-operation

      - name: Another task in block
        debug:
          msg: "Block succeeded"

    rescue:
      - name: This runs if block fails
        debug:
          msg: "Block failed, running rescue"

      - name: Send alert
        mail:
          to: admin@example.com
          subject: "Playbook failure"
          body: "Error: {{ ansible_failed_result.msg }}"

    always:
      - name: This always runs (cleanup)
        debug:
          msg: "Always runs regardless of success/failure"
```

### `any_errors_fatal`

```yaml
- hosts: all
  any_errors_fatal: true   # Stop play for ALL hosts if any host fails
  tasks:
    - name: Critical task
      command: /usr/bin/critical
```

### `max_fail_percentage`

```yaml
- hosts: webservers
  max_fail_percentage: 30   # Abort if more than 30% of hosts fail
  tasks:
    - name: Deploy app
      command: /usr/bin/deploy
```

---

## 17. Modules (Key Ones)

### File & Directory

```yaml
# file — manage files/directories/symlinks
- file:
    path: /opt/myapp
    state: directory      # file, directory, link, absent, touch
    owner: www-data
    group: www-data
    mode: '0755'
    recurse: yes          # for directories

# copy — copy files to remote
- copy:
    src: files/app.conf
    dest: /etc/app/app.conf
    backup: yes           # backup existing file
    validate: '/usr/bin/app --check %s'

# fetch — fetch files from remote to control node
- fetch:
    src: /var/log/app.log
    dest: /tmp/logs/
    flat: yes

# template — deploy Jinja2 templates
- template:
    src: templates/app.conf.j2
    dest: /etc/app/app.conf

# lineinfile — manage single lines in files
- lineinfile:
    path: /etc/hosts
    line: '192.168.1.10 myapp.local'
    state: present

# blockinfile — manage blocks of text in files
- blockinfile:
    path: /etc/hosts
    block: |
      192.168.1.10 web1
      192.168.1.11 web2
    marker: "# {mark} ANSIBLE MANAGED BLOCK"

# stat — get file info
- stat:
    path: /etc/nginx/nginx.conf
  register: file_info

# find — find files
- find:
    paths: /var/log
    patterns: "*.log"
    age: "7d"
    recurse: yes
  register: old_logs
```

### Package Management

```yaml
# apt (Debian/Ubuntu)
- apt:
    name:
      - nginx
      - git
      - python3
    state: present         # present, absent, latest
    update_cache: yes
    cache_valid_time: 3600 # Only update cache if older than 1hr

# yum (RHEL/CentOS 7)
- yum:
    name: httpd
    state: latest

# dnf (RHEL/CentOS 8+, Fedora)
- dnf:
    name: httpd
    state: present

# package (cross-platform)
- package:
    name: git
    state: present

# pip
- pip:
    name: flask
    version: "2.0.1"
    virtualenv: /opt/myapp/venv
```

### Service Management

```yaml
- service:
    name: nginx
    state: started       # started, stopped, restarted, reloaded
    enabled: yes         # start on boot

# systemd-specific
- systemd:
    name: myapp
    state: started
    enabled: yes
    daemon_reload: yes   # reload systemd daemon first
```

### User & Group

```yaml
- user:
    name: devops
    state: present
    uid: 1010
    group: developers
    groups:
      - sudo
      - docker
    append: yes           # don't remove from other groups
    shell: /bin/bash
    home: /home/devops
    create_home: yes
    password: "{{ 'password' | password_hash('sha512') }}"

- group:
    name: developers
    state: present
    gid: 1100
```

### Command Execution

```yaml
# command — run a command (no shell features)
- command: /usr/bin/app --init
  args:
    chdir: /opt/myapp     # change dir before running
    creates: /opt/myapp/.initialized  # skip if file exists

# shell — run via shell (supports pipes, redirects, etc.)
- shell: "grep -r 'error' /var/log/ | wc -l"
  register: error_count

# raw — run raw SSH command (no Python needed)
- raw: yum install -y python3

# script — run a local script on remote
- script: scripts/setup.sh arg1 arg2
  args:
    creates: /opt/app/.initialized
```

### Networking

```yaml
# uri — make HTTP requests
- uri:
    url: "https://api.example.com/health"
    method: GET
    status_code: 200
    headers:
      Authorization: "Bearer {{ api_token }}"
  register: api_response

# wait_for — wait for condition
- wait_for:
    host: db.example.com
    port: 3306
    state: started
    timeout: 60

- wait_for:
    path: /tmp/app.ready
    state: present

# wait_for_connection
- wait_for_connection:
    timeout: 300
```

### Git & Archives

```yaml
# git
- git:
    repo: https://github.com/example/myapp.git
    dest: /opt/myapp
    version: main          # branch, tag, or commit hash
    force: yes

# unarchive — extract archives
- unarchive:
    src: /tmp/app.tar.gz   # local file
    dest: /opt/app
    remote_src: yes        # file is already on remote

# get_url — download a file
- get_url:
    url: "https://example.com/app-v1.0.tar.gz"
    dest: /tmp/app.tar.gz
    checksum: "sha256:abc123..."
```

### Cloud (AWS example)

```yaml
# Requires boto3 and botocore installed
# ec2_instance
- ec2_instance:
    name: my-web-server
    key_name: my-keypair
    instance_type: t3.micro
    image_id: ami-0abcd1234
    region: us-east-1
    state: present

# s3_object
- s3_object:
    bucket: my-bucket
    object: /path/to/file.txt
    src: /local/file.txt
    mode: put
```

---

## 18. Ansible Galaxy

Ansible Galaxy is a hub for finding, sharing, and downloading community roles and collections.

### Collections vs Roles

| | Roles | Collections |
|---|---|---|
| Contains | Tasks, handlers, templates | Roles, modules, plugins, playbooks |
| Install | `ansible-galaxy role install` | `ansible-galaxy collection install` |
| Namespace | N/A | `namespace.collection` |
| Modern | Legacy | Current standard |

### Working with Roles

```bash
# Search for roles
ansible-galaxy search nginx

# Install a role
ansible-galaxy role install geerlingguy.nginx

# Install specific version
ansible-galaxy role install geerlingguy.nginx,3.1.0

# List installed roles
ansible-galaxy role list

# Remove a role
ansible-galaxy role remove geerlingguy.nginx

# Install from requirements file
ansible-galaxy role install -r requirements.yml
```

### Working with Collections

```bash
# Install a collection
ansible-galaxy collection install community.general

# Install AWS collection
ansible-galaxy collection install amazon.aws

# List installed collections
ansible-galaxy collection list

# Install from requirements.yml
ansible-galaxy collection install -r requirements.yml
```

### `requirements.yml` File

```yaml
---
roles:
  - name: geerlingguy.nginx
    version: "3.1.0"
  - name: geerlingguy.mysql
  - src: https://github.com/user/role.git
    scm: git
    version: main

collections:
  - name: community.general
    version: ">=5.0.0"
  - name: amazon.aws
    version: "3.5.0"
  - name: ansible.posix
```

---

## 19. Dynamic Inventory

Dynamic inventory queries external systems (AWS, GCP, Azure, etc.) to generate inventory at runtime.

### AWS EC2 Dynamic Inventory

```bash
# Install AWS collection
ansible-galaxy collection install amazon.aws

# Create inventory file: aws_ec2.yml
```

```yaml
# aws_ec2.yml
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
  - us-west-2

filters:
  instance-state-name: running
  tag:Environment: production

keyed_groups:
  - key: tags.Role
    prefix: role
  - key: placement.region
    prefix: aws_region

hostnames:
  - private-ip-address
  - public-ip-address
  - dns-name

compose:
  ansible_host: public_ip_address
  ansible_user: "'ubuntu'"
```

```bash
# Test dynamic inventory
ansible-inventory -i aws_ec2.yml --list
ansible-inventory -i aws_ec2.yml --graph

# Use in playbook
ansible-playbook -i aws_ec2.yml playbook.yml
```

### Custom Dynamic Inventory Script

```python
#!/usr/bin/env python3
import json
import sys

def get_inventory():
    return {
        "webservers": {
            "hosts": ["192.168.1.10", "192.168.1.11"],
            "vars": {"http_port": 80}
        },
        "_meta": {
            "hostvars": {
                "192.168.1.10": {"ansible_user": "ubuntu"}
            }
        }
    }

if __name__ == "__main__":
    if len(sys.argv) == 2 and sys.argv[1] == '--list':
        print(json.dumps(get_inventory()))
    elif len(sys.argv) == 3 and sys.argv[1] == '--host':
        print(json.dumps({}))
    else:
        sys.exit(1)
```

```bash
chmod +x inventory.py
ansible-playbook -i inventory.py playbook.yml
```

---

## 20. Ansible Tower / AWX

**Ansible Tower** (commercial) and **AWX** (open-source) provide a web UI, REST API, and enterprise features on top of Ansible.

### Key Features

| Feature | Description |
|---|---|
| **Web UI** | Visual dashboard for managing playbooks and jobs |
| **RBAC** | Role-Based Access Control for users/teams |
| **Job Scheduling** | Cron-like scheduling for playbooks |
| **Workflow** | Multi-playbook pipelines with conditions |
| **Credentials** | Centralized secure credential storage |
| **Notifications** | Email/Slack/webhook alerts on job status |
| **REST API** | Fully manageable via API |
| **Audit Logs** | Full history of who ran what, when |

### Core AWX/Tower Concepts

- **Organization** — Top-level grouping
- **Project** — Link to SCM (Git) containing playbooks
- **Inventory** — Hosts and groups
- **Credentials** — SSH keys, vault passwords, cloud credentials
- **Job Template** — Defines what playbook runs on what inventory
- **Job** — A single execution of a Job Template
- **Workflow** — Chain of Job Templates
- **Schedule** — Time-based triggers for Job Templates

### Install AWX (Docker)

```bash
git clone https://github.com/ansible/awx.git
cd awx
# Follow setup instructions in INSTALL.md
```

---

## 21. Best Practices

### Project Structure

```
my-ansible-project/
├── ansible.cfg
├── inventory/
│   ├── production/
│   │   ├── hosts.ini
│   │   ├── group_vars/
│   │   └── host_vars/
│   └── staging/
│       ├── hosts.ini
│       ├── group_vars/
│       └── host_vars/
├── group_vars/
│   └── all/
│       ├── vars.yml
│       └── vault.yml
├── playbooks/
│   ├── site.yml        # Master playbook
│   ├── webservers.yml
│   └── dbservers.yml
├── roles/
│   ├── common/
│   ├── nginx/
│   └── mysql/
├── files/
├── templates/
├── vars/
└── README.md
```

### Key Best Practices

1. **Use roles** for reusability — avoid monolithic playbooks.
2. **Name every task** — descriptive names make debugging easier.
3. **Use `defaults/main.yml`** for role variables so they can be overridden easily.
4. **Store secrets in Vault** — never commit plain-text secrets.
5. **Use `--check` mode** before running in production (`ansible-playbook --check`).
6. **Use `--diff`** to see what would change before applying.
7. **Idempotency** — design tasks to be safely re-runnable.
8. **Use `changed_when: false`** for read-only commands.
9. **Prefer YAML dict syntax** over `key=value` in ad-hoc commands and tasks.
10. **Pin versions** in `requirements.yml` for reproducible installs.
11. **Use tags** for selective execution.
12. **Separate environments** — use different inventory directories for prod/staging.
13. **Keep playbooks in version control** (Git).
14. **Lint playbooks** with `ansible-lint` before committing.

```bash
# Install and run ansible-lint
pip install ansible-lint
ansible-lint playbook.yml
```

---

## 22. Interview Q&A Cheatsheet

### Fundamentals

**Q: What is Ansible and why is it agentless?**  
A: Ansible is an open-source automation tool. It's agentless because it uses existing SSH (Linux) or WinRM (Windows) protocols to communicate with managed nodes — no software needs to be installed on target machines.

**Q: What is idempotency in Ansible?**  
A: A task is idempotent if running it multiple times produces the same result as running it once. E.g., installing a package — if it's already installed, Ansible won't reinstall it; it just reports `ok`.

**Q: Difference between `command` and `shell` modules?**  
A: `command` executes a command directly without invoking a shell — it doesn't support pipes, redirects, or shell variables. `shell` invokes `/bin/sh` and supports full shell features like pipes (`|`), redirects (`>`), and glob expansion (`*`). Prefer `command` for security when shell features aren't needed.

**Q: What is the difference between `vars` and `defaults` in a role?**  
A: `defaults/main.yml` has the **lowest precedence** — any inventory variable, playbook variable, or `-e` flag will override it. `vars/main.yml` has **higher precedence** and is harder to override. Use `defaults` for values users should feel free to customize, and `vars` for values the role logic depends on.

**Q: What is `gather_facts` and when would you disable it?**  
A: `gather_facts: true` (default) runs the `setup` module at play start to collect system info. Disabling it (`gather_facts: false`) speeds up playbook execution when you don't need system facts.

**Q: How does Ansible handle errors?**  
A: By default, Ansible stops executing on a host if a task fails. You can use `ignore_errors: yes`, `failed_when`, `block/rescue/always`, `any_errors_fatal`, or `max_fail_percentage` to customize error behavior.

**Q: What is `register` and `when` and how do they work together?**  
A: `register` captures task output into a variable. `when` conditionally runs a task based on an expression. Together they let you branch logic: run a command, check its output, then conditionally run another task.

**Q: Explain Ansible Vault.**  
A: Ansible Vault encrypts sensitive data (passwords, keys) using AES-256. You can encrypt whole files or individual strings. Encrypted files are safe to commit to version control. Vault password is provided at runtime via `--ask-vault-pass` or a password file.

**Q: What are handlers and when do they run?**  
A: Handlers are tasks triggered by `notify` from another task. They run **at the end of the play**, not immediately when notified. If a handler is notified multiple times, it still runs only once.

**Q: What is the difference between `include_tasks` and `import_tasks`?**  

| | `import_tasks` | `include_tasks` |
|---|---|---|
| Processing | Static (at parse time) | Dynamic (at runtime) |
| Can use `when` on include | Yes (applied to all tasks) | Yes (evaluated at runtime) |
| Can use `loop` on include | No | Yes |
| Tags | Applied to all included tasks | Must tag the include itself |

**Q: What is a collection in Ansible?**  
A: Collections are the modern packaging format in Ansible. They bundle roles, modules, plugins, and playbooks into a distributable unit. Installed via `ansible-galaxy collection install namespace.collection`.

**Q: How do you speed up Ansible playbook execution?**  
A: 
- Disable `gather_facts` when not needed
- Increase `forks` in `ansible.cfg` (default: 5)
- Use `strategy: free` to let faster hosts proceed without waiting
- Enable SSH pipelining (`pipelining = True` in `ansible.cfg`)
- Use `async` and `poll` for long-running tasks
- Use `--limit` to target fewer hosts
- Cache facts (`fact_caching`)

**Q: What is `delegate_to`?**  
A: It runs a task on a different host than the current target. Common use: run a load balancer command from a specific host, or create a local file on the control node.

```yaml
- name: Remove from load balancer
  command: lb-remove {{ inventory_hostname }}
  delegate_to: loadbalancer.example.com
```

**Q: What is `serial` in a playbook?**  
A: `serial` controls how many hosts Ansible processes at once during a rolling update.

```yaml
- hosts: webservers
  serial: 2        # Process 2 hosts at a time
  # or
  serial: "25%"    # Process 25% of hosts at a time
```

---

*📌 Good luck with your interview! These notes cover 95%+ of Ansible topics asked in DevOps interviews.*
