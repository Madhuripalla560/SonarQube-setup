# SonarQube Local Setup on Ubuntu Linux

## Step 1: Update the System
Update the package index and upgrade existing packages to ensure your system is up to date.
```bash
sudo apt update
sudo apt upgrade -y
```

## Step 2: Install Java
SonarQube requires Java to run. We'll use OpenJDK 17, which is supported by the latest SonarQube versions.
```bash
sudo apt install openjdk-17-jdk -y
java -version
```

## Step 3: Install PostgreSQL
SonarQube uses a database to store data. PostgreSQL is recommended.

### 3.1 Install Dependencies
```bash
sudo apt install curl ca-certificates -y
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
```

### 3.2 Add PostgreSQL Repository
```bash
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
sudo apt update
```

### 3.3 Install PostgreSQL 15
```bash
sudo apt install postgresql-15 -y
```

### 3.4 Configure PostgreSQL
Switch to the `postgres` user and create a dedicated database and user for SonarQube.
```bash
sudo -i -u postgres
createuser sonar
createdb sonar -O sonar
psql
ALTER USER sonar WITH ENCRYPTED PASSWORD 'your_password';
\q
exit
```

## Step 4: Install SonarQube
Download and configure SonarQube.

### 4.1 Download SonarQube
```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.5.1.90531.zip
```

### 4.2 Extract and Move SonarQube
```bash
unzip sonarqube-10.5.1.90531.zip
sudo mv sonarqube-10.5.1.90531 /opt/sonarqube
```

### 4.3 Create a SonarQube User
```bash
sudo adduser --system --no-create-home --group --disabled-login sonarqube
sudo chown -R sonarqube:sonarqube /opt/sonarqube
```

### 4.4 Configure SonarQube Database Connection
```bash
sudo vi /opt/sonarqube/conf/sonar.properties
```
Add or update the following lines:
```
sonar.jdbc.username=sonar
sonar.jdbc.password=your_password
sonar.jdbc.url=jdbc:postgresql://localhost/sonar
```

## Step 5: Create a Systemd Service
This allows SonarQube to run as a service and start automatically.

### 5.1 Create the Service File
```bash
sudo vi /etc/systemd/system/sonarqube.service
```
Add the following content:
```
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonarqube
Group=sonarqube
Restart=always
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

### 5.2 Start and Enable the Service
```bash
sudo systemctl daemon-reload
sudo systemctl start sonarqube
sudo systemctl enable sonarqube
```

## Step 6: Configure System Limits
These limits are required for SonarQube to function correctly.

### 6.1 Increase File Descriptors and Process Limits
```bash
sudo vi /etc/security/limits.conf
```
Add:
```
sonarqube - nofile 65536
sonarqube - nproc 4096
```

### 6.2 Set Virtual Memory Limits
```bash
sudo sysctl -w vm.max_map_count=262144
sudo vi /etc/sysctl.conf
```
Add:
```
vm.max_map_count=262144
```
Apply changes:
```bash
sudo sysctl -p
```

## Final Step: Access SonarQube
Once SonarQube is running, access it via a web browser:
```
http://<your-server-ip>:9000
```
Default login credentials:
```
Username: admin
Password: admin
```
**Important:** Change the default password after your first login.
