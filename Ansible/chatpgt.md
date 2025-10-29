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
