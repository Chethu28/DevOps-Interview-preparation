---

## 🔐 1. Using **Ansible Vault** (Built-in and most common)

**Ansible Vault** allows you to **encrypt sensitive data** (like variables, files, or entire playbooks).

### ✅ Use Cases:

* Encrypting variables files (e.g., `vault.yml`)
* Encrypting specific variables inline
* Encrypting entire playbooks or inventory files

### 🔧 Example:

**Step 1:** Create an encrypted secrets file

```bash
ansible-vault create secrets.yml
```

This will open a text editor; enter your sensitive data:

```yaml
db_user: admin
db_password: My$ecretP@ssword
```

**Step 2:** Use the secrets file in your playbook

```yaml
# playbook.yml
---
- hosts: all
  vars_files:
    - secrets.yml
  tasks:
    - name: Print DB user
      debug:
        msg: "Database user is {{ db_user }}"
```

**Step 3:** Run the playbook with the vault password

```bash
ansible-playbook playbook.yml --ask-vault-pass
```

Or specify a password file (recommended for automation):

```bash
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass.txt
```

---

### 🔁 Common Vault Commands

| Command                          | Description                   |
| -------------------------------- | ----------------------------- |
| `ansible-vault create file.yml`  | Create and encrypt a new file |
| `ansible-vault edit file.yml`    | Edit an encrypted file        |
| `ansible-vault view file.yml`    | View an encrypted file        |
| `ansible-vault encrypt file.yml` | Encrypt an existing file      |
| `ansible-vault decrypt file.yml` | Decrypt an encrypted file     |
| `ansible-vault rekey file.yml`   | Change the vault password     |

---

### 💡 Partial Encryption (Only encrypt specific variables)

You can encrypt only certain values instead of the whole file:

```bash
ansible-vault encrypt_string 'SuperSecret123' --name 'db_password'
```

Output:

```yaml
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          32666334393566393539346136623332623238333032643733653463393763333835643461303730
          6433303639333836643534653538643263343866616539610a323761306263343137353835643762
          35373738313334383265376338386337613863323931343938643732323734356337636530383031
          6533376234346132630a323066643161613637663332306631393163316137306565383431303737
          3435
```

Paste this directly in your YAML.

---

## 🔑 2. Using Environment Variables

If you don't want to store secrets in files, you can use environment variables dynamically.

### Example:

```bash
export DB_USER=admin
export DB_PASS=MySecretPass
```

In your playbook:

```yaml
tasks:
  - name: Show DB user
    debug:
      msg: "DB user is {{ lookup('env', 'DB_USER') }}"
```

🧩 Works great for CI/CD pipelines (Jenkins, GitLab, GitHub Actions, etc.) where secrets are injected as environment variables.

---

## ☁️ 3. Using **External Secret Managers**

You can integrate Ansible with **HashiCorp Vault**, **AWS Secrets Manager**, or **Azure Key Vault**.

---

### 🏗️ Example — HashiCorp Vault Integration

Install the required plugin:

```bash
pip install hvac
```

**vault.yml**

```yaml
---
- hosts: localhost
  vars:
    vault_addr: "http://127.0.0.1:8200"
    vault_token: "{{ lookup('env', 'VAULT_TOKEN') }}"
  tasks:
    - name: Get secret from HashiCorp Vault
      set_fact:
        db_password: "{{ lookup('hashi_vault', 'secret=secret/data/myapp:password token=' + vault_token + ' url=' + vault_addr) }}"

    - debug:
        msg: "The password is {{ db_password }}"
```

---

### ☁️ AWS Secrets Manager Integration

Install boto3 and required collection:

```bash
pip install boto3 botocore
ansible-galaxy collection install community.aws
```

**Example Playbook:**

```yaml
- hosts: localhost
  tasks:
    - name: Retrieve secret from AWS Secrets Manager
      community.aws.aws_secret:
        name: my-app-secret
        region: ap-south-1
      register: secret_data

    - debug:
        msg: "Secret value is {{ secret_data.secret_string }}"
```

---

### ☁️ Azure Key Vault Integration

```yaml
- hosts: localhost
  vars:
    azure_keyvault_name: myKeyVault
    secret_name: dbPassword
  tasks:
    - name: Get secret from Azure Key Vault
      azure.azcollection.azure_rm_keyvaultsecret_info:
        vault_uri: "https://{{ azure_keyvault_name }}.vault.azure.net/"
        name: "{{ secret_name }}"
      register: secret_result

    - debug:
        msg: "Secret value is {{ secret_result.secrets[0].value }}"
```

---

## ⚙️ 4. Combining Vault + Environment Variables (Best Practice)

Use **Ansible Vault** for encryption and **environment variables** in pipelines for flexibility:

```yaml
db_user: "{{ lookup('env', 'DB_USER') | default('admin') }}"
db_password: "{{ vault_db_password }}"
```

Vault file:

```yaml
vault_db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          ...
```

---

## 🧠 Best Practices

✅ Never hardcode secrets directly in playbooks.
✅ Store vault passwords securely (e.g., Jenkins credentials store).
✅ Rotate vault passwords periodically using `ansible-vault rekey`.
✅ Use different vaults for different environments (e.g., `dev`, `prod`).
✅ Use dynamic secret fetch from external stores for cloud-native infra.

-------------------------------------------------------------------------------------------------------------------------------------

-------------------------------------------------------------------------------------------------------------------------------------

# 🧩 What is an Ansible Role?

An **Ansible Role** is a way to **organize and modularize your playbooks** into reusable, maintainable, and shareable components.

Think of a **role** as a **complete automation unit** that handles a specific function — like installing Nginx, configuring users, or deploying an app.

Instead of writing one long playbook, you split it into **roles**, each containing **tasks**, **handlers**, **templates**, **files**, and **variables**.

---

# 🏗️ Role Directory Structure

When you create a role (e.g., `webserver`), Ansible expects a specific folder structure:

```
roles/
  webserver/
    tasks/
      main.yml           # Contains all main tasks
    handlers/
      main.yml           # Contains handlers (like service restarts)
    templates/
      index.html.j2      # Jinja2 templates (optional)
    files/
      myconfig.conf      # Files to copy (optional)
    vars/
      main.yml           # Role-specific variables
    defaults/
      main.yml           # Default (lowest priority) variables
    meta/
      main.yml           # Role dependencies
```

---

# ⚙️ Creating a Role

You can create a new role manually or with Ansible’s command-line utility.

```bash
ansible-galaxy init webserver
```

Output:

```
- webserver was created successfully
```

This creates the folder structure shown above.

---

# 🧠 Role Example — Install and Configure Nginx

### Step 1: Create Role

```bash
ansible-galaxy init webserver
```

---

### Step 2: Edit `roles/webserver/tasks/main.yml`

```yaml
---
- name: Install Nginx
  ansible.builtin.yum:
    name: nginx
    state: present

- name: Copy Nginx config
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart Nginx

- name: Start Nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: yes
```

---

### Step 3: Edit `roles/webserver/handlers/main.yml`

```yaml
---
- name: Restart Nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
```

---

### Step 4: Add a Template `roles/webserver/templates/nginx.conf.j2`

```jinja
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

http {
  server {
    listen 80;
    location / {
      root /usr/share/nginx/html;
      index index.html;
    }
  }
}
```

---

### Step 5: Use Role in a Playbook

**`site.yml`**

```yaml
---
- hosts: webservers
  become: yes
  roles:
    - webserver
```

Run:

```bash
ansible-playbook site.yml -i inventory
```

---

# 🎯 Why Use Roles?

| Benefit            | Description                                                |
| ------------------ | ---------------------------------------------------------- |
| ✅ **Reusability**  | Roles can be reused across multiple playbooks or projects. |
| 🧹 **Modularity**  | Makes large playbooks easier to manage.                    |
| 🔄 **Consistency** | Standard structure keeps teams aligned.                    |
| 🚀 **Shareable**   | Roles can be shared via **Ansible Galaxy**.                |
| 🔧 **Extensible**  | You can include dependencies and variables.                |

---

# 🔁 Including Roles in Playbooks

You can include roles in multiple ways:

### 1️⃣ Simple way

```yaml
roles:
  - webserver
```

### 2️⃣ With variables

```yaml
roles:
  - role: webserver
    vars:
      nginx_port: 8080
```

### 3️⃣ Conditional execution

```yaml
- hosts: all
  roles:
    - { role: webserver, when: ansible_os_family == 'RedHat' }
```

---

# ⚡ Role Dependencies

If your role depends on another role (e.g., installing `common` before `webserver`), define it in:
**`roles/webserver/meta/main.yml`**

```yaml
---
dependencies:
  - role: common
```

---

# 🧩 Variable Precedence Inside Roles

| Level | Directory                      | Priority        |
| ----- | ------------------------------ | --------------- |
| 1️⃣   | `defaults/main.yml`            | Lowest priority |
| 2️⃣   | `vars/main.yml`                | Higher priority |
| 3️⃣   | Playbook variables, extra-vars | Highest         |

So you should define **defaults** for optional values and **vars** for required, role-specific configurations.

---

# 🌍 Using Roles from **Ansible Galaxy**

Ansible Galaxy is a **public repository** for sharing roles.

Search for existing roles:

```bash
ansible-galaxy search nginx
```

Install a role:

```bash
ansible-galaxy install geerlingguy.nginx
```

Use it in your playbook:

```yaml
- hosts: webservers
  roles:
    - geerlingguy.nginx
```

---

# 🧠 Real-time DevOps Example

In real-world CI/CD pipelines, you’ll often structure Ansible like this:

```
playbooks/
  site.yml
  deploy.yml
roles/
  common/
  webserver/
  database/
  application/
inventory/
  dev
  prod
```

Each environment (dev, prod) uses the same roles but different vars:

```
group_vars/dev.yml
group_vars/prod.yml
```

---

# 🛠️ Interview Tip

✅ **Q:** Why do we use roles instead of plain playbooks?
**A:** Roles bring modularity, reusability, and maintainability to large automation projects, helping DevOps teams scale infrastructure as code efficiently.

✅ **Q:** What’s the difference between `vars` and `defaults` in roles?
**A:** `defaults` have the lowest precedence (can be overridden easily), while `vars` have higher precedence (harder to override).

✅ **Q:** How can you pass variables to a role dynamically?
**A:** Using `-e` (extra-vars) or within the playbook under `vars:`.

---

---

# 🚀 What is Dynamic Inventory in Ansible?

By default, Ansible uses a **static inventory** (like `hosts` or `inventory.ini`), where you manually define servers:

```ini
[webservers]
192.168.1.10
192.168.1.11
```

But in real-world **cloud environments** (AWS, Azure, GCP, Kubernetes, etc.), infrastructure changes dynamically — instances come and go.

👉 To handle this, we use a **Dynamic Inventory**, which automatically queries cloud APIs or other sources to discover hosts.

---

# 🧠 Key Idea

Dynamic inventories **generate inventory data at runtime** (instead of hardcoding IPs).

They can be implemented using:

1. **Inventory plugins** (YAML-based, built-in) ✅ recommended
2. **External scripts** (custom Python, shell, JSON output)
3. **Cloud inventory integrations** (AWS, Azure, GCP, VMware, etc.)

---

# 🧩 1️⃣ Example: AWS Dynamic Inventory using built-in plugin

Ansible provides ready-made inventory plugins for all major clouds.

## 📦 Install dependencies

```bash
pip install boto3 botocore
ansible-galaxy collection install amazon.aws
```

## 📁 Create Inventory File

Let’s call it: `aws_ec2.yml`

```yaml
plugin: amazon.aws.ec2
regions:
  - ap-south-1
filters:
  instance-state-name: running
keyed_groups:
  - key: tags.Role
    prefix: tag
hostnames:
  - private-ip-address
compose:
  ansible_host: private_ip_address
```

👉 This tells Ansible to:

* Use the AWS EC2 inventory plugin
* Fetch running EC2 instances from **ap-south-1**
* Group them based on **tags**
* Use **private IPs** for SSH

---

## 🔑 AWS Authentication

Ansible uses your existing AWS credentials:

* From environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`)
* Or from `~/.aws/credentials`
* Or IAM roles (if running inside AWS)

Example:

```bash
export AWS_ACCESS_KEY_ID=AKIA...
export AWS_SECRET_ACCESS_KEY=secretkey...
```

---

## ✅ Test Dynamic Inventory

```bash
ansible-inventory -i aws_ec2.yml --graph
```

Output (example):

```
@all:
  |--@tag_WebServer:
  |  |--10.0.1.12
  |--@tag_Database:
  |  |--10.0.2.21
```

---

## 🧰 Use it in a Playbook

**`site.yml`**

```yaml
---
- hosts: tag_WebServer
  become: yes
  tasks:
    - name: Check uptime
      command: uptime
```

Run:

```bash
ansible-playbook -i aws_ec2.yml site.yml
```

🎉 Ansible automatically discovers all EC2 instances with tag `Role=WebServer` and runs the playbook.

---

# 🧩 2️⃣ Example: Custom Dynamic Inventory Script (Python)

You can also create your **own dynamic inventory** — useful when integrating with APIs, databases, or internal systems.

Let’s create `dynamic_inventory.py`.

```python
#!/usr/bin/env python3
import json

inventory = {
    "webservers": {
        "hosts": ["192.168.1.10", "192.168.1.11"],
        "vars": {"ansible_user": "ec2-user"}
    },
    "dbservers": {
        "hosts": ["192.168.1.12"],
        "vars": {"ansible_user": "ubuntu"}
    },
    "_meta": {
        "hostvars": {
            "192.168.1.10": {"env": "dev"},
            "192.168.1.12": {"env": "prod"}
        }
    }
}

print(json.dumps(inventory))
```

Make it executable:

```bash
chmod +x dynamic_inventory.py
```

Test it:

```bash
./dynamic_inventory.py
```

Output:

```json
{
  "webservers": {
    "hosts": ["192.168.1.10", "192.168.1.11"],
    "vars": {"ansible_user": "ec2-user"}
  },
  "dbservers": {
    "hosts": ["192.168.1.12"],
    "vars": {"ansible_user": "ubuntu"}
  },
  "_meta": {
    "hostvars": {
      "192.168.1.10": {"env": "dev"},
      "192.168.1.12": {"env": "prod"}
    }
  }
}
```

Use it as inventory:

```bash
ansible-inventory -i dynamic_inventory.py --graph
```

Output:

```
@all:
  |--@webservers:
  |  |--192.168.1.10
  |  |--192.168.1.11
  |--@dbservers:
  |  |--192.168.1.12
```

Now you can run:

```bash
ansible-playbook -i dynamic_inventory.py site.yml
```

---

# 🧩 3️⃣ Example: Dynamic Inventory for Kubernetes

If your infrastructure runs in **Kubernetes**, Ansible can query pods dynamically.

```yaml
plugin: kubernetes.core.k8s
connections:
  - kubeconfig: ~/.kube/config
    namespaces:
      - default
keyed_groups:
  - key: labels.app
    prefix: app
```

Run:

```bash
ansible-inventory -i k8s.yml --graph
```

---

# 🧠 Real-time Use Case (DevOps Example)

In CI/CD pipelines (like Jenkins, GitHub Actions, GitLab):

* The inventory file (`aws_ec2.yml`) dynamically fetches running EC2 servers
* Ansible deploys to only those servers tagged with `Environment=Prod`
* No need to update static IP lists manually

---

# ⚙️ Best Practices

✅ Use **inventory plugins** over custom scripts (easier and maintained by Ansible).
✅ Keep credentials out of inventory — use environment variables or vault.
✅ Group hosts logically (by tag, environment, region).
✅ Use `ansible-inventory --graph` or `--list` for troubleshooting.
✅ Combine with **Ansible Vault** to encrypt sensitive connection data.

---

# 🧩 Quick Summary Table

| Method                   | Description                  | Example File             |
| ------------------------ | ---------------------------- | ------------------------ |
| Static Inventory         | Manually defined hosts       | `inventory.ini`          |
| Dynamic Inventory Script | Custom script returns JSON   | `dynamic_inventory.py`   |
| Inventory Plugin         | YAML file using cloud plugin | `aws_ec2.yml`, `k8s.yml` |
| External Source          | CMDB, REST API, etc.         | via lookup plugin        |

---

Would you like me to show a **real-time DevOps project example** that uses:

* AWS dynamic inventory (`aws_ec2.yml`)
* Tags-based host targeting
* Vault for secret management
* And a CI/CD Jenkinsfile that runs the playbook dynamically?

-----------------------------------------------------------------------------------------------

---

# ⚙️ What is an Ansible Module?

> 🔹 A **module** in Ansible is a **unit of work** — a reusable piece of code that performs a specific task on a managed node.
>
> Every task in your playbook **calls a module** under the hood.

Example:

```yaml
- name: Install nginx
  ansible.builtin.yum:
    name: nginx
    state: present
```

Here, `yum` is a **module** that installs packages.

---

# 🧩 Types of Ansible Modules Used in Real-Time Projects

Ansible has **thousands** of modules — grouped by category.
Let’s focus on the **most-used ones in real-world DevOps automation**.

---

## 🏗️ 1. **System Modules**

Used for managing users, groups, packages, services, files, and processes on servers.

| Module                        | Purpose                 | Example                                                    |
| ----------------------------- | ----------------------- | ---------------------------------------------------------- |
| `ansible.builtin.yum` / `apt` | Install/remove packages | `yum: name=httpd state=present`                            |
| `ansible.builtin.service`     | Manage services         | `service: name=nginx state=restarted`                      |
| `ansible.builtin.user`        | Manage user accounts    | `user: name=devops state=present`                          |
| `ansible.builtin.group`       | Manage groups           | `group: name=admins state=present`                         |
| `ansible.builtin.file`        | Manage files/dirs       | `file: path=/opt/app mode=0755 state=directory`            |
| `ansible.builtin.copy`        | Copy local files        | `copy: src=config.conf dest=/etc/config.conf`              |
| `ansible.builtin.template`    | Deploy templates        | `template: src=app.conf.j2 dest=/etc/app.conf`             |
| `ansible.builtin.lineinfile`  | Modify file lines       | `lineinfile: path=/etc/hosts line='10.0.0.1 app'`          |
| `ansible.builtin.command`     | Run system command      | `command: uptime`                                          |
| `ansible.builtin.shell`       | Run shell commands      | `shell: "systemctl status nginx"`                          |
| `ansible.builtin.cron`        | Manage cron jobs        | `cron: name="backup" minute=0 hour=2 job="/opt/backup.sh"` |

🔹 **Real-time Use Case:**

```yaml
- name: Configure Web Server
  hosts: web
  become: yes
  tasks:
    - yum: name=httpd state=present
    - service: name=httpd state=started enabled=yes
    - copy: src=index.html dest=/var/www/html/index.html
```

---

## ☁️ 2. **Cloud Modules**

Used for provisioning and managing cloud infrastructure (AWS, Azure, GCP, etc.)

| Cloud | Module                              | Purpose                      |
| ----- | ----------------------------------- | ---------------------------- |
| AWS   | `amazon.aws.ec2`                    | Create EC2 instances         |
| AWS   | `amazon.aws.s3_bucket`              | Create/manage S3 buckets     |
| AWS   | `amazon.aws.ec2_group`              | Manage Security Groups       |
| AWS   | `amazon.aws.cloudformation`         | Deploy CloudFormation stacks |
| Azure | `azure.azcollection.azure_rm_vm`    | Create Azure VMs             |
| GCP   | `google.cloud.gcp_compute_instance` | Manage Compute Engine VMs    |

🔹 **Real-time Example (AWS):**

```yaml
- name: Launch EC2 instance
  hosts: localhost
  gather_facts: no
  tasks:
    - amazon.aws.ec2:
        key_name: dev-key
        instance_type: t2.micro
        image_id: ami-0abcdef123456
        wait: yes
        count: 2
        region: ap-south-1
```

---

## 🐳 3. **Container Modules**

Used for managing Docker or Podman containers and images.

| Module                               | Purpose                  | Example                                                  |
| ------------------------------------ | ------------------------ | -------------------------------------------------------- |
| `community.docker.docker_container`  | Manage Docker containers | `docker_container: name=nginx image=nginx state=started` |
| `community.docker.docker_image`      | Manage Docker images     | `docker_image: name=nginx source=pull`                   |
| `containers.podman.podman_container` | Manage Podman containers | `podman_container: name=app image=busybox`               |

🔹 **Real-time Example:**

```yaml
- name: Deploy Nginx Container
  hosts: docker
  become: yes
  tasks:
    - community.docker.docker_container:
        name: nginx
        image: nginx:latest
        state: started
        ports:
          - "80:80"
```

---

## ☸️ 4. **Kubernetes Modules**

Used to manage K8s clusters, pods, deployments, and services.

| Module                     | Purpose                     |
| -------------------------- | --------------------------- |
| `kubernetes.core.k8s`      | Create/update K8s resources |
| `kubernetes.core.k8s_info` | Get K8s object info         |
| `kubernetes.core.helm`     | Deploy Helm charts          |

🔹 **Real-time Example:**

```yaml
- name: Deploy App to Kubernetes
  hosts: localhost
  tasks:
    - kubernetes.core.k8s:
        state: present
        definition:
          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: nginx
          spec:
            replicas: 2
            selector:
              matchLabels:
                app: nginx
            template:
              metadata:
                labels:
                  app: nginx
              spec:
                containers:
                  - name: nginx
                    image: nginx:latest
                    ports:
                      - containerPort: 80
```

---

## 🗄️ 5. **Database Modules**

Used for managing databases (MySQL, PostgreSQL, MongoDB, etc.)

| Module                                 | Purpose                     |
| -------------------------------------- | --------------------------- |
| `community.mysql.mysql_db`             | Manage MySQL databases      |
| `community.mysql.mysql_user`           | Manage MySQL users          |
| `community.postgresql.postgresql_db`   | Manage PostgreSQL databases |
| `community.postgresql.postgresql_user` | Manage PostgreSQL users     |

🔹 **Real-time Example:**

```yaml
- name: Create MySQL DB and user
  hosts: db
  become: yes
  tasks:
    - community.mysql.mysql_db:
        name: myappdb
        state: present
    - community.mysql.mysql_user:
        name: appuser
        password: "securePass"
        priv: "myappdb.*:ALL"
        state: present
```

---

## 🧩 6. **Networking Modules**

Used for network device configuration and management.

| Vendor  | Module                               | Purpose                   |
| ------- | ------------------------------------ | ------------------------- |
| Cisco   | `cisco.ios.ios_config`               | Manage IOS device configs |
| Juniper | `junipernetworks.junos.junos_config` | Manage JunOS configs      |
| Arista  | `arista.eos.eos_config`              | Manage EOS devices        |

🔹 Example:

```yaml
- name: Push config to Cisco device
  hosts: routers
  connection: network_cli
  tasks:
    - cisco.ios.ios_config:
        lines:
          - hostname router1
          - interface Gi0/1
          - description Uplink
```

---

## 💾 7. **Files & Storage Modules**

Manage file systems, mounts, partitions, and permissions.

| Module                    | Purpose                | Example                                                     |
| ------------------------- | ---------------------- | ----------------------------------------------------------- |
| `ansible.builtin.mount`   | Mount partitions       | `mount: path=/data src=/dev/sdb1 fstype=ext4 state=mounted` |
| `ansible.builtin.fetch`   | Copy files from remote | `fetch: src=/var/log/messages dest=/tmp/logs`               |
| `ansible.builtin.archive` | Compress files         | `archive: path=/data dest=/backup/data.tgz`                 |

---

## 🔐 8. **Security Modules**

Used for key management, secret management, and permissions.

| Module                              | Purpose                 |
| ----------------------------------- | ----------------------- |
| `ansible.builtin.authorized_key`    | Manage SSH keys         |
| `ansible.builtin.seboolean`         | Manage SELinux booleans |
| `ansible.builtin.ufw` / `firewalld` | Configure firewalls     |

🔹 Example:

```yaml
- name: Add SSH key for user
  authorized_key:
    user: devops
    key: "{{ lookup('file', '/home/devops/.ssh/id_rsa.pub') }}"
```

---

## 🧰 9. **Utility Modules**

Used for debugging, variable manipulation, and control flow.

| Module                          | Purpose                     | Example                                         |
| ------------------------------- | --------------------------- | ----------------------------------------------- |
| `ansible.builtin.debug`         | Print debug info            | `debug: msg="Host is {{ inventory_hostname }}"` |
| `ansible.builtin.set_fact`      | Set custom variables        | `set_fact: app_env=prod`                        |
| `ansible.builtin.include_tasks` | Include external task files | `include_tasks: common.yml`                     |
| `ansible.builtin.import_role`   | Import a role dynamically   | `import_role: name=webserver`                   |
| `ansible.builtin.pause`         | Pause playbook execution    | `pause: seconds=10`                             |

---

# 🧠 Real-Time Project Example

Here’s how a **production-grade playbook** uses multiple modules together:

```yaml
- name: Deploy full web stack
  hosts: web
  become: yes
  tasks:
    - name: Install packages
      yum:
        name: ['nginx', 'git']
        state: present

    - name: Create app user
      user:
        name: appuser
        shell: /bin/bash

    - name: Clone code from GitHub
      git:
        repo: https://github.com/demo/webapp.git
        dest: /opt/webapp

    - name: Deploy configuration
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

✅ Uses modules: `yum`, `user`, `git`, `template`, `service`

---

# 🏁 Summary: Most Common Modules in Interviews

| Category   | Most Asked Modules                                        |
| ---------- | --------------------------------------------------------- |
| System     | `yum`, `apt`, `service`, `copy`, `template`, `lineinfile` |
| Cloud      | `amazon.aws.ec2`, `azure.azcollection.azure_rm_vm`        |
| Containers | `docker_container`, `docker_image`                        |
| Kubernetes | `kubernetes.core.k8s`                                     |
| Database   | `mysql_db`, `mysql_user`                                  |
| Utility    | `debug`, `set_fact`, `include_tasks`, `pause`             |

---

---

# ⚙️ 🎯 Goal

> ✅ You want to **skip a specific task** — even if it fails — and allow the **rest of the playbook to continue running**.

By default, Ansible **stops execution** when any task fails.

To handle this gracefully, you can use:

* `ignore_errors: yes`
* `failed_when: false`
* `block` + `rescue` blocks
* `when:` condition
* Custom handlers to control logic

---

## 🧩 1️⃣ Method 1 — `ignore_errors: yes`  ✅ *(Most Common)*

This is the simplest and most widely used method.

It tells Ansible:

> “Even if this task fails, don’t stop — just move to the next task.”

### 🔧 Example:

```yaml
- name: Create directory (ignore if already exists)
  file:
    path: /tmp/mydir
    state: directory
  ignore_errors: yes

- name: Continue with next task
  debug:
    msg: "Next task executed even though previous failed"
```

✅ **Result:**
If `/tmp/mydir` creation fails, Ansible logs the error but **continues execution**.

---

## 🧩 2️⃣ Method 2 — `failed_when: false`  🧠 *(Mark failure as success)*

Sometimes, a task fails with a **non-zero exit code**, but that failure is actually **acceptable** in your logic.

You can override the failure condition using `failed_when`.

### 🔧 Example:

```yaml
- name: Check if package is installed
  shell: "rpm -q nginx"
  register: check_nginx
  failed_when: false

- debug:
    msg: "Output was {{ check_nginx.stdout }}"
```

✅ **What happens:**

* Even if `rpm -q nginx` fails (exit code ≠ 0),
  Ansible won’t mark it as failed.
* You can then handle it manually in later tasks.

---

## 🧩 3️⃣ Method 3 — Combine `ignore_errors` + `register`  🔄

If you want to **capture** the error but **skip it safely**, use `register` and `ignore_errors` together.

### 🔧 Example:

```yaml
- name: Try to restart a non-existent service
  service:
    name: fake_service
    state: restarted
  register: result
  ignore_errors: yes

- debug:
    msg: "Service restart failed but continuing. Error: {{ result.msg }}"
```

✅ This logs the error message and continues the playbook execution.

---

## 🧩 4️⃣ Method 4 — Using `block`, `rescue`, and `always`  🧱

This is more **structured error handling**, like `try/catch` in programming.

### 🔧 Example:

```yaml
- block:
    - name: Try risky task
      command: /bin/false

    - name: This will be skipped if above fails
      debug:
        msg: "This runs only if previous succeeds"

  rescue:
    - name: Handle failure
      debug:
        msg: "Previous task failed, skipping gracefully"

  always:
    - name: Always execute cleanup
      debug:
        msg: "Cleanup actions executed"
```

✅ **Behavior:**

* If `/bin/false` fails → `rescue` block runs.
* `always` block runs at the end no matter what.

Perfect for **rollback or cleanup logic** in production pipelines.

---

## 🧩 5️⃣ Method 5 — Conditional Execution (`when:`) 🧩

You can **skip** a task **based on a condition**, e.g., skip if previous task failed.

### 🔧 Example:

```yaml
- name: Run risky command
  shell: "exit 1"
  register: result
  ignore_errors: yes

- name: Skip this task if previous failed
  debug:
    msg: "Previous succeeded"
  when: result is succeeded
```

✅ Only runs if previous task was successful.

---

## 🧩 6️⃣ Method 6 — Using `any_errors_fatal: false` in Playbook

If you want your **entire playbook** to continue even if some hosts or tasks fail:

```yaml
- hosts: all
  any_errors_fatal: false
  tasks:
    - name: Risky task
      command: /bin/false
      ignore_errors: yes
```

---

# 🧠 Real-Time Use Case Example

Imagine you’re deploying an app, and you want to:

* Try restarting a service.
* If it fails, skip and send a log message.
* Continue deployment.

```yaml
- name: Deploy Application
  hosts: web
  become: yes
  tasks:
    - name: Try restarting nginx
      service:
        name: nginx
        state: restarted
      register: nginx_result
      ignore_errors: yes

    - name: Log service restart failure
      debug:
        msg: "Warning: nginx restart failed — {{ nginx_result.msg }}"
      when: nginx_result is failed

    - name: Continue with application deployment
      git:
        repo: "https://github.com/demo/app.git"
        dest: /opt/app
```

✅ Here:

* Playbook won’t stop if nginx fails.
* Logs the failure.
* Continues to deploy code.

---

# 🧩 7️⃣ (Bonus) — Use `max_fail_percentage`

In multi-host runs, you can control how many host failures are acceptable:

```yaml
- hosts: all
  max_fail_percentage: 30
  tasks:
    - name: Test
      command: /bin/false
      ignore_errors: yes
```

If more than 30% of hosts fail → playbook stops.

---

# ✅ Summary

| Method                                     | Description                       | Real-time Use                 |
| ------------------------------------------ | --------------------------------- | ----------------------------- |
| `ignore_errors: yes`                       | Skips failure, continues playbook | Most common                   |
| `failed_when: false`                       | Forces success condition          | Accept non-critical failures  |
| `block` + `rescue`                         | Structured error handling         | Rollback/cleanup scenarios    |
| `when:`                                    | Conditional execution             | Skip based on previous status |
| `register` + `ignore_errors`               | Capture failure details           | Log/notify failures           |
| `any_errors_fatal` / `max_fail_percentage` | Control global fail behavior      | Multi-host orchestration      |

---

### 💬 Interview Tip:

> **Q:** How do you make Ansible skip a failing task but still record its result?
> **A:** Use `ignore_errors: yes` along with `register` to capture and log the failure, ensuring the playbook continues gracefully.

---

---

# 🧠 **Concept Overview**

In Ansible:

* **Control Node (Master):**
  The system where you install and run Ansible (usually your jump/bastion or CI/CD runner).
* **Managed Nodes (Workers):**
  The target servers where Ansible executes tasks over **SSH**.

Ansible is **agentless**, meaning — no agent is installed on worker machines; it only needs **Python + SSH access**.

---

# ⚙️ **1️⃣ Prerequisites**

You’ll need:

* One **Control Node (Master)**: e.g., `ansible-master`
* One or more **Managed Nodes (Workers)**: e.g., `ansible-node1`, `ansible-node2`
* All systems should have:

  * Linux OS
  * Python installed (usually pre-installed)
  * SSH connectivity between Master → Worker

---

# 🧩 **2️⃣ Step-by-Step Configuration**

---

## **Step 1: Install Ansible on the Master Node**

### 🔧 On Control Node:

```bash
sudo apt update -y           # For Ubuntu/Debian
sudo apt install ansible -y
```

Or for CentOS/RHEL:

```bash
sudo yum install epel-release -y
sudo yum install ansible -y
```

Check version:

```bash
ansible --version
```

---

## **Step 2: Create SSH Key on Master Node**

SSH key allows passwordless communication to worker nodes.

### 🔧 Generate key:

```bash
ssh-keygen
```

(Press Enter for all prompts — it creates `/home/<user>/.ssh/id_rsa` and `.pub`)

---

## **Step 3: Copy SSH Key to Worker Nodes**

Send the public key to each worker machine:

```bash
ssh-copy-id <username>@<worker-node-ip>
```

Example:

```bash
ssh-copy-id ubuntu@192.168.1.11
ssh-copy-id ubuntu@192.168.1.12
```

Now test passwordless login:

```bash
ssh ubuntu@192.168.1.11
```

If you log in **without a password**, you’re good ✅

---

## **Step 4: Create the Inventory File**

Inventory defines all managed nodes.

By default: `/etc/ansible/hosts`
Or you can create a custom one in your project, e.g. `inventory.ini`.

### 🔧 Example `/etc/ansible/hosts`:

```ini
[webservers]
192.168.1.11 ansible_user=ubuntu
192.168.1.12 ansible_user=ubuntu

[dbservers]
192.168.1.13 ansible_user=ubuntu
```

Or using hostnames:

```ini
[webservers]
web1 ansible_host=192.168.1.11 ansible_user=ubuntu
web2 ansible_host=192.168.1.12 ansible_user=ubuntu
```

---

## **Step 5: Test the Connection**

Run a simple ping test to check connectivity:

```bash
ansible all -m ping -i /etc/ansible/hosts
```

✅ Expected output:

```yaml
192.168.1.11 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

If all nodes respond with `pong`, your Master–Worker setup is successful 🎉

---

# 🧩 **6️⃣ Optional: Ansible Configuration File**

You can create a project-level config file: `ansible.cfg`

```ini
[defaults]
inventory = ./inventory.ini
remote_user = ubuntu
host_key_checking = False
retry_files_enabled = False
```

Now you can run commands without `-i` flag.

---

# 🧩 **7️⃣ Verify Managed Nodes**

To see the inventory list recognized by Ansible:

```bash
ansible-inventory --list -y
```

Or in a tree view:

```bash
ansible-inventory --graph
```

---

# 🧩 **8️⃣ Run Your First Ad-hoc Command**

```bash
ansible all -m shell -a "uptime"
ansible webservers -m yum -a "name=httpd state=present" --become
```

✅ This installs Apache on all webservers from the Master Node.

---

# 🧠 **Real-Time Notes & Best Practices**

| Area                       | Best Practice                                                                                                      |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **SSH Auth**               | Always use key-based authentication, not passwords.                                                                |
| **Privilege Escalation**   | Use `become: yes` for tasks needing root privileges.                                                               |
| **Inventory Organization** | Group servers logically (e.g., `[appservers]`, `[dbservers]`).                                                     |
| **Connectivity**           | Ensure Python 3 is installed on managed nodes (`yum install python3 -y`).                                          |
| **Security**               | Restrict SSH to Ansible Master IP only.                                                                            |
| **Dynamic Inventories**    | For cloud setups (AWS, Azure, GCP), use dynamic inventory scripts or plugins (we can cover that next if you want). |

---

# 🧩 **9️⃣ Real-Time Directory Layout Example**

A best-practice Ansible project structure:

```
ansible-project/
├── ansible.cfg
├── inventory.ini
├── playbooks/
│   ├── site.yml
│   ├── webserver.yml
│   └── dbserver.yml
└── roles/
    ├── web/
    │   ├── tasks/
    │   ├── handlers/
    │   ├── templates/
    │   └── vars/
```

---

# ✅ **Summary**

| Component                      | Role                                             |
| ------------------------------ | ------------------------------------------------ |
| **Master Node (Control Node)** | Where Ansible is installed and playbooks are run |
| **Worker Node (Managed Node)** | Target servers managed via SSH                   |
| **Inventory File**             | Lists and groups managed hosts                   |
| **SSH Keys**                   | Enable agentless, passwordless control           |
| **Ansible Modules**            | Execute actions on managed nodes                 |
| **Playbooks**                  | Define automation workflows                      |

---

### 💬 **Interview Tip:**

> **Q:** How does Ansible communicate with managed nodes?
> **A:** Over SSH (agentless). The control node pushes modules (in Python) to the managed node, executes them, and removes them after execution.

---

---

# 🧭 **🔹 ANSIBLE COMMAND CHEAT SHEET (For Interviews + Real-Time Use)**

---

## ⚙️ 1️⃣ **Inventory & Host Verification Commands**

| Command                           | Description                               | Example Output / Usage            |
| --------------------------------- | ----------------------------------------- | --------------------------------- |
| `ansible --version`               | Check Ansible version and config paths    | Verifies installation             |
| `ansible all --list-hosts`        | Lists all hosts in inventory              | Shows count and hostnames         |
| `ansible-inventory --list -y`     | Displays detailed inventory data (YAML)   | Useful to debug dynamic inventory |
| `ansible-inventory --graph`       | Shows inventory tree view                 | Great for topology visualization  |
| `ansible webservers --list-hosts` | Lists all servers in group `[webservers]` | Confirms group members            |

📘 *Interview Tip:*

> Q: “How do you verify that your Ansible inventory file is properly configured?”
> A: “Use `ansible-inventory --list -y` or `--graph` to validate host groups and variables.”

---

## ⚙️ 2️⃣ **Connectivity Testing**

| Command                     | Purpose                                           | Example                      |
| --------------------------- | ------------------------------------------------- | ---------------------------- |
| `ansible all -m ping`       | Checks SSH connectivity to all nodes              | `"ping": "pong"`             |
| `ansible dbservers -m ping` | Pings only dbservers group                        | Checks if Python/SSH work    |
| `ansible all -m setup`      | Collects all system facts (hardware, OS, network) | Very useful for dynamic vars |

📘 *Real-time Tip:*
`setup` module helps debug fact gathering issues and verify OS types dynamically.

---

## ⚙️ 3️⃣ **Ad-hoc Commands**

Used for **quick one-liners** — no playbook needed.

| Action           | Command                                                                  |
| ---------------- | ------------------------------------------------------------------------ |
| Check uptime     | `ansible all -m command -a "uptime"`                                     |
| Check disk usage | `ansible all -a "df -h"`                                                 |
| Check memory     | `ansible all -m shell -a "free -m"`                                      |
| Restart service  | `ansible webservers -m service -a "name=httpd state=restarted" --become` |
| Install package  | `ansible webservers -m yum -a "name=httpd state=present" --become`       |
| Copy file        | `ansible all -m copy -a "src=/tmp/file dest=/tmp/"`                      |
| Delete file      | `ansible all -m file -a "path=/tmp/testfile state=absent"`               |

📘 *Interview Tip:*

> “What’s the difference between `command` and `shell` modules?”
> ✅ `command` doesn’t process pipes/redirection; `shell` runs inside a shell (allows `|`, `>`, etc.)

---

## ⚙️ 4️⃣ **Playbook Execution Commands**

| Command                                                     | Description                          |
| ----------------------------------------------------------- | ------------------------------------ |
| `ansible-playbook site.yml`                                 | Run playbook                         |
| `ansible-playbook site.yml -i inventory.ini`                | Specify custom inventory             |
| `ansible-playbook site.yml --check`                         | Dry-run (no changes made)            |
| `ansible-playbook site.yml --diff`                          | Show what changed in files/templates |
| `ansible-playbook site.yml -v`                              | Verbose output (single level)        |
| `ansible-playbook site.yml -vvv`                            | Debug-level verbose                  |
| `ansible-playbook site.yml --limit webservers`              | Run only on specific group           |
| `ansible-playbook site.yml --start-at-task="Install nginx"` | Resume from a specific task          |
| `ansible-playbook site.yml --tags "deploy"`                 | Run only tasks with specific tag     |
| `ansible-playbook site.yml --skip-tags "config"`            | Skip tasks with specific tag         |

📘 *Real-time Tip:*
Always test playbooks with `--check` (dry-run) before actual execution in production.

---

## ⚙️ 5️⃣ **Ansible Facts & Variables**

| Command                                                        | Description                        |
| -------------------------------------------------------------- | ---------------------------------- |
| `ansible all -m setup`                                         | Show all host facts                |
| `ansible all -m setup -a "filter=ansible_hostname"`            | Get specific fact                  |
| `ansible localhost -m debug -a "var=hostvars['web1']"`         | Access variables from another host |
| `ansible-playbook play.yml --extra-vars "env=dev version=1.0"` | Pass runtime variables             |
| `ansible-playbook play.yml -e "@vars.json"`                    | Load external variable file        |

📘 *Interview Tip:*

> “How do you pass dynamic variables at runtime?”
> ✅ Use `--extra-vars` (or `-e`) flag.

---

## ⚙️ 6️⃣ **Tags for Selective Task Execution**

| Command                                                            | Use                    |
| ------------------------------------------------------------------ | ---------------------- |
| Add tag in playbook: `tags: install`                               | Assigns tag to a task  |
| Run tagged task: `ansible-playbook play.yml --tags "install"`      | Runs only tagged tasks |
| Skip tagged task: `ansible-playbook play.yml --skip-tags "config"` | Skips certain tasks    |

📘 *Real-time Tip:*
Tags are lifesavers for CI/CD pipelines — let you rerun only parts of playbooks.

---

## ⚙️ 7️⃣ **Debugging and Error Handling**

| Command                                    | Description                                |
| ------------------------------------------ | ------------------------------------------ |
| `ansible-playbook play.yml -vvv`           | Detailed debugging output                  |
| `ansible-playbook play.yml --syntax-check` | Check YAML syntax before running           |
| `ansible-playbook play.yml --list-tasks`   | Lists all tasks before execution           |
| `ansible-playbook play.yml --list-tags`    | Lists all tags                             |
| `ansible-playbook play.yml --step`         | Step-by-step confirmation before each task |

📘 *Interview Tip:*

> “How do you debug a failing playbook?”
> ✅ Use `-vvv` and `--step` to trace task execution and variables.

---

## ⚙️ 8️⃣ **Role Management Commands**

| Command                                          | Description                |
| ------------------------------------------------ | -------------------------- |
| `ansible-galaxy init role_name`                  | Creates new role skeleton  |
| `ansible-galaxy list`                            | Lists installed roles      |
| `ansible-galaxy install geerlingguy.apache`      | Install role from Galaxy   |
| `ansible-galaxy remove role_name`                | Remove role                |
| `ansible-playbook play.yml --roles-path ./roles` | Use custom roles directory |

📘 *Real-time Tip:*
Keep reusable configurations as roles (e.g., `nginx`, `java`, `docker`).

---

## ⚙️ 9️⃣ **Dynamic Inventory Commands**

| Command                                    | Description                                            |
| ------------------------------------------ | ------------------------------------------------------ |
| `ansible-inventory -i aws_ec2.yml --graph` | View AWS dynamic inventory groups                      |
| `ansible all -m ping -i aws_ec2.yml`       | Ping AWS EC2 hosts dynamically                         |
| `ansible-inventory -i <plugin>.yml --list` | List hosts fetched from plugin (AWS, Azure, GCP, etc.) |

📘 *Interview Tip:*

> “How does Ansible automatically discover hosts from AWS?”
> ✅ Use dynamic inventory plugins like `aws_ec2`, configured in `.yml`.

---

## ⚙️ 🔟 **Privilege Escalation (Become)**

| Command                                                | Example                            |
| ------------------------------------------------------ | ---------------------------------- |
| `ansible all -m command -a "cat /etc/shadow" --become` | Run task as root                   |
| `ansible-playbook play.yml --become --ask-become-pass` | Ask for sudo password              |
| Add inside playbook: `become: yes`                     | Run all tasks with sudo privileges |

📘 *Real-time Tip:*
Most production playbooks use `become: yes` since tasks need root privileges.

---

## ⚙️ 11️⃣ **Vault (Secrets Encryption)**

| Command                                                           | Description                      |
| ----------------------------------------------------------------- | -------------------------------- |
| `ansible-vault create secrets.yml`                                | Create encrypted file            |
| `ansible-vault view secrets.yml`                                  | View encrypted file              |
| `ansible-vault edit secrets.yml`                                  | Edit encrypted file              |
| `ansible-vault encrypt file.yml`                                  | Encrypt existing file            |
| `ansible-vault decrypt file.yml`                                  | Decrypt file                     |
| `ansible-playbook play.yml --ask-vault-pass`                      | Run playbook with vault password |
| `ansible-playbook play.yml --vault-password-file .vault_pass.txt` | Use stored password              |

📘 *Interview Tip:*

> “How do you store sensitive data like passwords or API keys in Ansible?”
> ✅ “Use Ansible Vault encryption with `ansible-vault create/encrypt`.”

---

## ⚙️ 12️⃣ **Miscellaneous Commands**

| Command               | Use                                      |
| --------------------- | ---------------------------------------- |
| `ansible-config list` | List all configuration options           |
| `ansible-config dump` | Show current configuration values        |
| `ansible-doc -l`      | List all available modules               |
| `ansible-doc yum`     | Show documentation for specific module   |
| `ansible-console`     | Interactive shell to run ad-hoc commands |

📘 *Pro Tip:*
`ansible-doc` is a hidden gem — gives examples and parameters for every module.

---

# 🚀 **🎯 Most Frequently Asked in Interviews**

| Category     | Commands to Remember                            |
| ------------ | ----------------------------------------------- |
| Connectivity | `ansible all -m ping`                           |
| Ad-hoc       | `ansible all -m shell -a "uptime"`              |
| Playbook     | `ansible-playbook play.yml --check`             |
| Debug        | `ansible-playbook play.yml -vvv`                |
| Inventory    | `ansible-inventory --list -y`                   |
| Tags         | `--tags`, `--skip-tags`                         |
| Variables    | `--extra-vars "key=value"`                      |
| Vault        | `ansible-vault encrypt/decrypt`                 |
| Roles        | `ansible-galaxy init`, `ansible-galaxy install` |

---

# ✅ **Bonus: Real-Time Command Flow Example**

### Deploying Nginx on Web Servers:

```bash
ansible all -m ping
ansible webservers -m yum -a "name=nginx state=present" --become
ansible webservers -m service -a "name=nginx state=started" --become
ansible webservers -m shell -a "systemctl status nginx"
```

---
