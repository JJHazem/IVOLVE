# 🛠️ **Lab 8: Ansible Playbooks for Web Server and MySQL Configuration**

**Author:** Ahmed Hazem

## 📌 **Objective:**

Write Ansible playbooks to automate the configuration of a web server and MySQL database.

---

## 🚀 **Step 1: Install Ansible (if not installed)**

```bash
sudo apt update
sudo apt install ansible -y
```

---

## 📜 **Step 2: Find Your Server IP**

Run the following command on your server VM to get the IP address:

```bash
ip a
```

Look for the `inet` address under your primary network interface (e.g., `enp1s0`).

You can also check your inventory file or cloud provider console.

---

## 📜 **Step 3: Create the Inventory File**

Create a file called `inventory` and add your server IP addresses:

```
[web_servers]
172.30.1.2

[db_servers]
172.30.1.2
```

Replace the IP with the correct one (`inet 172.30.1.2` from `enp1s0`).

If the playbook says "no hosts matched," double-check this step!

---

## 📜 **Step 4: Create the Web Server Playbook**

Create a file called `web-server.yml`:

```yaml
- name: Configure Web Server
  hosts: web_servers
  become: true

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Start and Enable Nginx
      service:
        name: nginx
        state: started
        enabled: true

    - name: Create index.html
      copy:
        dest: /var/www/html/index.html
        content: "<h1>Welcome to the Web Server!</h1>"
```

---

## ⚡ **Step 5: Run the Web Server Playbook**

```bash
ansible-playbook -i inventory web-server.yml
```

### ✅ **Verification:**

1. **Check Nginx status:**

```bash
sudo systemctl status nginx
```

2. **Test the web page:**

Open a browser and visit: `http://<server_ip>`

You should see the message: `Welcome to the Web Server!`

If not, check firewall settings:

```bash
sudo ufw allow 'Nginx Full'
```

3. **Check Open Ports:**

To verify if Nginx is listening on port 80:

```bash
sudo netstat -tuln | grep 80
```

Or, if `netstat` is not installed:

```bash
ss -tuln | grep 80
```

If nothing shows, make sure Nginx is running and no other service is blocking the port.

# 🔒 **Lab 9: MySQL Setup with Ansible Vault**

## 📌 **Objective:**

Install MySQL, create the `ivolve` database, add a user with all privileges, and use Ansible Vault for secure credentials.

---

## 🔑 **Step 1: Create the Vault File**

Create a file to store credentials:

```bash
ansible-vault create secrets.yml
```

Add content:

```yaml
mysql_root_password: your_root_password
mysql_user: ivolve_user
mysql_password: your_secure_password
```

Save and exit (use a password for the vault).

---

## 📜 **Step 2: Write the MySQL Playbook**

Create `mysql-setup.yml`:

```yaml
- name: Install and Configure MySQL
  hosts: db_servers
  become: true
  vars_files:
    - secrets.yml

  tasks:
    - name: Install MySQL
      apt:
        name: mysql-server
        state: present

    - name: Install PyMySQL
      apt:
        name: python3-pymysql
        state: present

    - name: Start and Enable MySQL
      service:
        name: mysql
        state: started
        enabled: true

    - name: Create MySQL Database
      mysql_db:
        name: ivolve
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"

    - name: Create MySQL User
      mysql_user:
        name: "{{ mysql_user }}"
        password: "{{ mysql_password }}"
        priv: 'ivolve.*:ALL'
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"
        host: '%'
```

⚠️ **Important:**

- If you see the error: `Access denied for user 'root'@'localhost'`, you might need to allow root login with a password. Try this:

```bash
sudo mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH 'mysql_native_password' BY '{{ mysql_root_password }}';
FLUSH PRIVILEGES;
exit
```

- Double-check the installation of `python3-pymysql` to avoid library errors.

---

## ⚡ **Step 3: Run the MySQL Playbook with Vault**

```bash
ansible-playbook -i inventory mysql-setup.yml --ask-vault-pass
```

You’ll be prompted to enter the vault password. If you get a "no hosts matched" error, verify your inventory file.

### ✅ **Verification:**

1. **Check MySQL service:**

```bash
sudo systemctl status mysql
```

2. **Connect to MySQL:**

```bash
mysql -u root -p
```

3. **Verify database and user:**

```sql
SHOW DATABASES;
SELECT user, host FROM mysql.user;
```

You should see the `ivolve` database and the `ivolve_user`.

---

## 🎯 **Results:**

- **Web Server:** Nginx installed and running with a basic HTML page.
- **Database:** MySQL installed, `ivolve` database created, user added with encrypted credentials.

**Author:** Ahmed Hazem

Let me know if you’d like me to refine this or add anything else! 🚀

