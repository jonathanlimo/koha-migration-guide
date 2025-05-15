```markdown
# Migrate & Upgrade Koha Server (22.11 to 24.11)

## **1. Using the Koha Packages**

### **Setting Up the Keys for Koha Package Sources**
```sh
sudo apt update
sudo apt install sudo apt-transport-https ca-certificates curl
sudo mkdir -p --mode=0755 /etc/apt/keyrings
sudo curl -fsSL https://debian.koha-community.org/koha/gpg.asc -o /etc/apt/keyrings/koha.asc
sudo apt update
```

### **Set Up the Package Sources**
```sh
echo 'deb [signed-by=/etc/apt/keyrings/koha.asc] https://debian.koha-community.org/koha 22.11 main' | sudo tee /etc/apt/sources.list.d/koha.list
sudo apt update
```

### **Install Koha**
```sh
sudo apt install koha-common
```

### **Install and Secure the Database**
```sh
sudo apt install mariadb-server
sudo mysql_secure_installation
sudo mariadb -u root -p
```
_(Answer "Y" to all prompts)_

### **Create Database User**
```sh
CREATE USER 'database_to_restore_user'@'%' IDENTIFIED BY 'database_to_restore_user_password';
CREATE USER 'database_to_restore_user'@'localhost' IDENTIFIED BY 'database_to_restore_user_password';

GRANT ALL PRIVILEGES ON *.* TO 'database_to_restore_user'@'%' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON `database_to_restore_user\_%`.* TO 'database_to_restore_user'@'%';
GRANT ALL PRIVILEGES ON `koha`.* TO 'database_to_restore_user'@'%';
GRANT ALL PRIVILEGES ON `koha`.* TO 'database_to_restore_user'@'localhost';

FLUSH PRIVILEGES;
EXIT;
```

## **2. Backup & Restore Koha**

### **Change Backup File Permissions**
```sh
ssh server_to_restore_ip
cd /var/spool/koha/instance_name/
sudo chmod 0777 instance_name.sql.gz
sudo chmod 0777 instance_name.tar.gz
exit
```

### **Copy Backup Files**
```sh
scp koha@server_to_restore_ip:/var/spool/koha/instance_name/instance_name.sql.gz .
scp koha@server_to_restore_ip:/var/spool/koha/instance_name/instance_name.tar.gz .
```

### **Restore Koha**
```sh
sudo koha-restore instance_name.sql.gz instance_name.tar.gz
```

## **3. Configure Apache**
### **Enable Apache Modules**
```sh
sudo a2enmod rewrite cgi headers proxy_http
sudo systemctl restart apache2
```

### **Set Apache Listening Port**
```sh
sudo nano /etc/apache2/ports.conf
```
_(Modify the file to include:)_
```
Listen 80
Listen <staff interface port number>
```
```sh
sudo systemctl restart apache2
```

## **4. Reindex Koha**

```sh
sudo koha-rebuild-zebra -f instance_name
```

### **Enable Plack**
```sh
sudo koha-plack --enable instance_name
sudo koha-plack --start instance_name
sudo systemctl restart apache2
```

---

### **5. Set Up the Package Sources to 23.11**
```sh
echo 'deb [signed-by=/etc/apt/keyrings/koha.asc] https://debian.koha-community.org/koha 23.11 main' | sudo tee /etc/apt/sources.list.d/koha.list
sudo apt update
sudo apt upgrade
```



### **6. Set Up the Package Sources 24.11**
```sh
echo 'deb [signed-by=/etc/apt/keyrings/koha.asc] https://debian.koha-community.org/koha 24.11 main' | sudo tee /etc/apt/sources.list.d/koha.list
sudo apt update
sudo apt upgrade
```

## **7. Final step - Reindex Koha**

```sh
sudo koha-rebuild-zebra -f instance_name
```
