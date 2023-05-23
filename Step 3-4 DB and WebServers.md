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

