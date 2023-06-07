> ## Prerequisites
> * Deploy a fully updated Ubuntu 20.04 LTS server with at least 2GB of RAM and 1 vCPU cores.
> * Create a non-root user with sudo access.
> 
> ## installation on server
> 
> 
> ### Install OpenJDK 11
> * SSH to your Ubuntu server as a non-root user with sudo access.
> * Install OpenJDK 11.
> ```
> sudo apt-get install openjdk-11-jdk -y
> ```
> 
> ### Install and Configure PostgreSQL
> 
> * Add the PostgreSQL repository.
> 
> ```
>  sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
>  ```
> * Add the PostgreSQL signing key.
> 
> ```
>  wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
> ```
> 
> * Install PostgreSQL.
> 
> ```
> sudo apt install postgresql postgresql-contrib -y
> ```
> 
> * Enable the database server to start automatically on reboot.
> ```
> sudo systemctl enable postgresql
> ```
> 
> * Start the database server.
> ```
>  sudo systemctl start postgresql
> ```
> 
> * Change the default PostgreSQL password.
> ```
> sudo passwd postgres
> ```
> * Switch to the postgres user.
> ```
>  su - postgres
> ```
> 
> * Create a user named sonar.
> 
> ```
> createuser sonar
> ```
> * Log in to PostgreSQL.
> ```
> psql
> ```
> * Set a password for the sonar user. Use a strong password in place of my_strong_password.
> ```
> ALTER USER sonar WITH ENCRYPTED password 'my_strong_password';
> ```
> * Create a sonarqube database and set the owner to sonar.
> ```
> CREATE DATABASE sonarqube OWNER sonar;
> ```
> * Grant all the privileges on the sonarqube database to the sonar user.
> ```
> GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar;
> ```
> 
> * Exit PostgreSQL.
> ```
> \q
> ```
> 
> * Return to your non-root sudo user account.
> ```
> exit
> ```
> 
> ### Download and Install SonarQube
> 
> * Install the zip utility, which is needed to unzip the SonarQube files.
> ```
> sudo apt-get install zip -y
> ```
> Locate the latest download URL from [the SonarQube official download page](https://www.sonarqube.org/downloads/ "Named link title")
> 
> * Download the SonarQube distribution files.
> ```
> sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-<VERSION_NUMBER>.zip
> ```
> * Unzip the downloaded file.
> ``
> sudo unzip sonarqube-<VERSION_NUMBER>.zip
> ``
> 
> * Move the unzipped files to /opt/sonarqube directory
> ```
> sudo mv sonarqube-<VERSION_NUMBER> /opt/sonarqube
> ```
> ### Add SonarQube Group and User
> Create a dedicated user and group for SonarQube, which can not run as the root user.
> 
> * Create a sonar group.
> ```
> sudo groupadd sonar
> ```
> * Create a sonar user and set /opt/sonarqube as the home directory.
> ```
> sudo useradd -d /opt/sonarqube -g sonar sonar
> ```
> * Grant the sonar user access to the /opt/sonarqube directory.
> ```
> sudo chown sonar:sonar /opt/sonarqube -R
> ```
> 
> ### Configure SonarQube
> * Edit the SonarQube configuration file.
> ```
> sudo nano /opt/sonarqube/conf/sonar.properties
> ```
> Find the following lines:
> ```
> #sonar.jdbc.username=
> #sonar.jdbc.password=
> ```
> * Uncomment the lines, and add the database user and password you created in Step 2.
> ```
> sonar.jdbc.username=sonar
> sonar.jdbc.password=my_strong_password
> ```
> * Below those two lines, add the sonar.jdbc.url.
> 
> ```
> sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
> ```
> 
> Save and exit the file.
> 
> * Edit the sonar script file.
> 
> ```
> sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh
> ```
> 
> * About 50 lines down, locate this line:
> ```
> #RUN_AS_USER=
> ```
> * Uncomment the line and change it to:
> ```
> RUN_AS_USER=sonar
> ```
> 
> * Save and exit the file.
> 
> ### Setup Systemd service
> * Create a systemd service file to start SonarQube at system boot.
> ```
> sudo nano /etc/systemd/system/sonar.service
> ```
> * Paste the following lines to the file.
> 
> ```
> [Unit]
> Description=SonarQube service
> After=syslog.target network.target
> 
> [Service]
> Type=forking
> 
> ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
> ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
> 
> User=sonar
> Group=sonar
> Restart=always
> 
> LimitNOFILE=65536
> LimitNPROC=4096
> 
> [Install]
> WantedBy=multi-user.target
> ```
> 
> Save and exit the file.
> 
> * Enable the SonarQube service to run at system startup.
> ```
>  sudo systemctl enable sonar
> ```
> 
> * Start the SonarQube service.
> ```
> sudo systemctl start sonar
> ```
> 
> * Check the service status.
> ```
> sudo systemctl status sonar
> ```
> 
> ### Modify Kernel System Limits
> SonarQube uses Elasticsearch to store its indices in an MMap FS directory. It requires some changes to the system defaults.
> 
> * Edit the sysctl configuration file.
> ```
> sudo nano /etc/sysctl.conf
> ```
> 
> * Add the following lines.
> ```
> vm.max_map_count=262144
> fs.file-max=65536
> ulimit -n 65536
> ulimit -u 4096
> ```
> Save and exit the file.
> 
> * Reboot the system to apply the changes.
> ```
> sudo reboot
> ```
> 
> ### Access SonarQube Web Interface
> * Access SonarQube in a web browser at your server's IP address on port 9000. For example:
> ```
> http://localhost:9000
> ```
