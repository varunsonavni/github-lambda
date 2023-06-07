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
> http://192.0.2.123:9000
> ```
> * Log in with username admin and password admin. SonarQube will prompt you to change your password. 





## Installation by Docker

> Installing SonarQube from the Docker Image
> 
> *Prerequisite* First you have to Docker on your Machine don't have you can install it by below command.
>        
> ```bash
>sudo apt-get install docker.io -y
>```

>Start the server by running:
>
>```bash
>docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest
>```
- Once your instance is up and running, Log in to http://localhost:9000 using System Administrator credentials:

  - login: admin    
  - password: admin

![Screenshot from 2022-07-07 12-22-49.png](images/Screenshot from 2022-07-07 12-22-49.png)

- Now here you can create new password:

![Screenshot from 2022-07-07 12-23-02.png](images/Screenshot from 2022-07-07 12-23-02.png)

- Here is your sonarqube server now you can create new project:

![Screenshot from 2022-07-07 12-23-29.png](images/Screenshot from 2022-07-07 12-23-29.png)


# SonarScanner

SonarQube Scanner / sonar-scanner - performs analysis and sends the results to SonarQube. It is a generic, CLI scanner, and you must provide explicit configurations that list the locations of your source files, test files, class files, ...

## Download and Install Sonar Scanner on Linux

Download the Sonarqube scanner package and move it to the OPT directory.

> - Make a directory /downloads/sonarqube
> 
> ```bash
> mkdir /downloads/sonarqube -p
> ```
> - You need to inside this  /downloads/sonarqube directory 
> 
> ```bash
> cd /downloads/sonarqube
> ```
> 
> - Download  sonar-scanner using wget command:
> 
> ```bash
> wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.2.0.1873-linux.zip
> ```
> 
> - Install unzip if you don’t have
> 
> ```bash
> sudo apt install unzip
> ```
> 
> - Unzip the file:
> 
> ```bash
> unzip sonar-scanner-cli-4.2.0.1873-linux.zip
> ```
> 
> - Move to /opt directory
> 
> ```bash
> mv sonar-scanner-4.2.0.1873-linux /opt/sonar-scanner
> ```
> 
> - Edit the sonar-scanner.properties file using below command:
> 
> ```bash
> vi /opt/sonar-scanner/conf/sonar-scanner.properties
> ```
> 
> - Add this line in this file:
> 
> ```bash
> sonar.host.url=http://localhost:9000
> sonar.sourceEncoding=UTF-8
> ```
> 
> - create a file to automate the required environment variables configuration
> 
> ```bash
> vi /etc/profile.d/sonar-scanner.sh
> ```
> 
> - Add this lines in this file
> 
> ```bash
> /bin/bash
> export PATH="$PATH:/opt/sonar-scanner/bin"
> ```
> 
> - Use the source command to add the sonar scanner command to the PATH variable:
> 
> ```bash
> source /etc/profile.d/sonar-scanner.sh
> ```
> 
> - To verify version of sonar-scanner
> 
> ```bash
> sonar-scanner -v
> ```

## Create sonar-project.properties in your repository
> 
> - Create a file in your repository with the name <b> sonar-project.properties </b> and add this lines into it
> 
> ```bah
> sonar.projectKey=Give-your-project-name
> sonar.qualitygate.wait=true
> ```
> 
> - Add SonarQube variables in your repository variables
> 
> ```bash
> SONAR_HOST_URL        : <<sonarqube-url>>
> SONAR_TOKEN           : <<sonarqube token>>
> ```
> ![image.png](https://cache404.net/wp-content/uploads/2020/07/sonarcloud-create-token.gif)

>

# Integration sonarqube with Gitlab

> - Update the following code into your <b> .gitlab-ci.yml </b> file in your repository.
> 
> 
> ```bash
> sonarqube-check:
>   stage: test
>   image: 
>     name: sonarsource/sonar-scanner-cli:latest
>     entrypoint: [""]
>   variables:
>     SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"  # Defines the location of the analysis task cache
>     GIT_DEPTH: "0"  # Tells git to fetch all the branches of the project, required by the analysis task
>   cache:
>     key: "${CI_JOB_NAME}"
>     paths:
>       - .sonar/cache
>   script: 
>     - sonar-scanner -Dsonar.projectKey=sample-nodejs-app -Dsonar.sources=. -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_TOKEN
>   allow_failure: true
>   only:
>     - main # or the name of your main branch
> ```

After running the pipeline go to <b> localhost:9000 </b> and you can check your project inside SonarQube and you can check your Code Quality & Code Security.

> ![image-1.png](images/image-1.png)

in a SonarQube you can create the user and assign the project so, User can see the project Code Quality & Code Security.

> ![image-2.png](images/image-2.png)




