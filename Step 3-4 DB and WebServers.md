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
   ```shell
   df -h
   sudo vi /etc/fstab
   ```
   Add the following line:
   ```text
   <NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
   ```

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
   setsebool -P httpd_execmem 1
   ```

7. Repeat steps 1-6 for the remaining 2 Web Servers.

