# Save the exact provided content into a Markdown file for GitHub use

content = """SonarQube local Setup on Ubuntu linux

Step 1: Update the System
1. Update the package list:

sudo apt update
Updates the package list from the Ubuntu repositories to ensure the latest information about available software.
2. Upgrade installed packages:

sudo apt upgrade -y
Upgrades all installed packages to their latest versions. The -y flag automatically accepts prompts.
________________________________________
Step 2: Install Java
1. Install OpenJDK 17:

sudo apt install openjdk-17-jdk -y
Installs the OpenJDK 17 development kit, which is compatible with SonarQube.
2. Verify the installation:

java -version
Confirms the installed Java version.
________________________________________
Step 3: Install PostgreSQL
1. Install Dependencies

sudo apt install curl ca-certificates -y
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
2. Add PostgreSQL Repository

sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
sudo apt update
3. Install PostgreSQL 15

sudo apt install postgresql-15 -y
4. Configure PostgreSQL
1. Switch to the postgres user:

sudo -i -u postgres
2. Create a user and database for SonarQube:

createuser sonar
createdb sonar -O sonar
3. Set a password for the sonar user:

psql
ALTER USER sonar WITH ENCRYPTED PASSWORD 'your_password';
\\q
4. Exit the postgres user:

exit
________________________________________
Step 4: Install SonarQube
1. Download SonarQube

wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.5.1.90531.zip
2. Extract and Move SonarQube

unzip sonarqube-10.5.1.90531.zip
sudo mv sonarqube-10.5.1.90531 /opt/sonarqube
3. Create a SonarQube User

sudo adduser --system --no-create-home --group --disabled-login sonarqube
sudo chown -R sonarqube:sonarqube /opt/sonarqube
4. Configure SonarQube
1. Edit the configuration file:

sudo vi /opt/sonarqube/conf/sonar.properties
2. Set the database properties:
sonar.jdbc.username=sonar
sonar.jdbc.password=your_password
sonar.jdbc.url=jdbc:postgresql://localhost/sonar
________________________________________
Step 5: Create a Systemd Service File
1. Create the Service File

sudo vi /etc/systemd/system/sonarqube.service
Add the following content:

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

2. Start and Enable the Service

sudo systemctl daemon-reload
sudo systemctl start sonarqube
sudo systemctl enable sonarqube
________________________________________
Step 6: Configure System Limits
1. Increase File Descriptors and Processes Limits
1. Edit the limits configuration file:

sudo vi /etc/security/limits.conf
2. Add the following lines:

sonarqube - nofile 65536
sonarqube - nproc 4096
2. Set Virtual Memory Limits
1. Set the vm.max_map_count:

sudo sysctl -w vm.max_map_count=262144
2. Make the change permanent:

sudo vi /etc/sysctl.conf
3. Add the following line:

vm.max_map_count=262144
4. Apply the changes:

sudo sysctl -p
________________________________________
Final Steps: Verify SonarQube Installation
1. Open your browser and navigate to:
http://<your-server-ip>:9000
2. Log in with the default credentials:
o Username: admin
o Password: admin
3. Change the default password upon first login.
"""

file_path = "/mnt/data/SonarQube_Ubuntu_Setup.md"
with open(file_path, "w") as f:
    f.write(content)

file_path
