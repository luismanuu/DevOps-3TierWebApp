# Step 3: Prepare DB Server

1. Launch and connect to the EC2 instance for the DB server.

2. Install MySQL and log in as the root user:
   ```shell
   sudo apt install mysql-server
   ```

3. Create the "tooling" database:
    ```shell
   sudo mysql
   ```
   ```mysql
   CREATE DATABASE tooling;
   ```

4. Create the "webaccess" user and grant them access to the "tooling" database:
   ```mysql
   CREATE USER 'webaccess'@'your_subnet_cidr' IDENTIFIED BY 'your_password';
   GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'your_subnet_cidr';
   FLUSH PRIVILEGES;
   ```
   Replace 'your_subnet_cidr' with the actual CIDR value of your webserver's subnet, and 'your_password' with a secure password for the "webaccess" user.

# Step 4: Prepare Web Servers

1. Launch 3 new EC2 instances with RHEL 9 Operating System. Important: Change NFS Security group, add the 3 private IP of webservers to connect to NFS ports

2. Install NFS client and mount the NFS server's export for apps:
   ```shell
   sudo yum install nfs-utils nfs4-acl-tools -y
   sudo mkdir /var/www
   sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
   ```

3. Verify the successful mount and configure it to persist after reboot:
![image](https://github.com/luismanuu/DevOps-3TierWebApp/assets/14170090/6b18cfe8-3a27-4775-8f71-0068dd62180a)

   ```shell
   df -h
   sudo vi /etc/fstab
   ```
   Add the following line:
   ```text
   <NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
   ```
   ![image](https://github.com/luismanuu/DevOps-3TierWebApp/assets/14170090/56c5347b-6adb-46bb-927c-7ea417b97b79)


4. Install Remi's repository, Apache, and PHP:
   ```shell
   sudo yum install httpd -y
   sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
   sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
   sudo dnf module reset php
   sudo dnf module enable php:remi-7.4
   sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
   ```

5. Start and enable PHP-FPM:
   ```shell
   sudo systemctl start php-fpm
   sudo systemctl enable php-fpm
   ```

6. Adjust SELinux boolean value for httpd:
   ```shell
   sudo setsebool -P httpd_execmem 1
   ```

7. Repeat steps 1-6 for the remaining 2 Web Servers.


8. Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. If you see the same files – it means NFS is mounted correctly. You can try to create a new file touch test.txt from one server and check if the same file is accessible from other Web Servers.

9. Mount the Apache log folder to the NFS server's export for logs. Follow Step 4 to ensure persistence after reboot.

10. Fork the tooling source code from Darey.io's GitHub Account to your own GitHub account.

11. Deploy the tooling website's code to the Web Servers, ensuring the html folder is placed in /var/www/html.

   Note 1: Open TCP port 80 on the Web Server.
   
   Note 2: If encountering a 403 Error, check /var/www/html folder permissions and disable SELinux with `sudo setenforce 0`. To make this change permanent, edit `/etc/sysconfig/selinux` and set `SELINUX=disabled`, then restart httpd.

12. Update the website's configuration in `/var/www/html/functions.php` to connect to the database.

13. Apply the `tooling-db.sql` script to the database using `mysql -h -u -p < tooling-db.sql`.

14. Create a new admin user in MySQL with username: myuser and password: password:
   ```sql
   INSERT INTO 'users' ('id', 'username', 'password', 'email', 'user_type', 'status')
   VALUES (1, 'myuser', '5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1');
   ```

15. Open the website in your browser (`http://<your-web-server-ip>/index.php`) and verify successful login with the myuser account.
```


