
## Install SonarQube:

SonarQube is an open-source platform for continuous code quality inspection. It helps developers analyze and improve code quality by detecting issues like bugs, vulnerabilities, code smells, and security flaws in projects.


#### Key Features of SonarQube:

- **Static Code Analysis** – Scans source code without executing it.
- **Detects Code Smells, Bugs & Vulnerabilities** – Improves maintainability and security.
- **Supports Multiple Languages** – Java, Python, JavaScript, C++, PHP, etc.
- **Integration with CI/CD Pipelines** – Works with Jenkins, GitHub Actions, GitLab CI/CD, etc.
- **Quality Gates** – Ensures code meets predefined standards before merging.
- **Custom Rules & Plugins** – Allows adding rules tailored to project needs.



### Prerequisites:

- SonarQube server requires Java version 11
- SonarQube scanners require Java version 11 or 17
- Maven or Gradle
- Database: PostgreSQL (9.6 or greater), MS SQL Server, or Oracle (12c/18c/19c)



### Install Java: 

```
yum install java-11-openjdk-devel -y

or,

apt install openjdk-11-jdk -y
```


```
java -version
```


```
echo $JAVA_HOME
```



###  Install Database:

#### For Postgres: 

```
psql -U postgres
```


```
create user sonar with encrypted password 'sonar123';

//create database sonarqubedb;

create database sonarqubedb owner sonar;

grant all privileges on database sonarqubedb to sonar;
```


```
psql -U sonar -d sonarqubedb
```


#### For MySQL: 

```
mysql -u root -p
```


```
CREATE USER 'sonar'@'localhost' IDENTIFIED WITH mysql_native_password BY 'sonar123';

CREATE DATABASE sonarqubedb;

GRANT ALL PRIVILEGES ON sonarqubedb.* TO 'sonar'@'localhost';

FLUSH PRIVILEGES;
```



### Add a new user:

```
useradd sonarqube
```


```
usermod -aG sudo sonarqube
usermod -aG wheel sonarqube
```


### Check the values with the following commands:


```
sysctl vm.max_map_count

    vm.max_map_count = 65530
```


```
sysctl fs.file-max

    fs.file-max = 173698
```


```
ulimit -n

    1024
```


```
ulimit -u

    6842
```


To set these values more permanently, you must update either `/etc/sysctl.d/99-sonarqube.conf` (or `/etc/sysctl.conf` as you wish) to reflect these values.


```
vim /etc/sysctl.conf

vm.max_map_count=524288
fs.file-max=131072
```


```
sysctl -p
```



If you're running SonarQube as a different user, replace `sonarqube` with that username. If the user running SonarQube (sonarqube in this example) does not have permission to have at least `131072` open descriptors, you must insert this line in `/etc/security/limits.d/99-sonarqube.conf` (or `/etc/security/limits.conf` as you wish):

- `istl0` : This specifies the username for which these limits apply.
- `-` : This means the limit applies to both **soft and hard limits** (instead of specifying soft or hard separately).
- `nofile 131072` : This sets the maximum number of open file descriptors (files that can be opened at the same time) to 131072.
- `nproc 8192` : This sets the maximum number of processes the user `istl0` can create to 8192.



```
vim /etc/security/limits.conf

#<userid> - nofile 65536
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
```


```
reboot
```


#### Verify the Changes:

```
ulimit -n
ulimit -u
```




### Download and Install SonarQube:


```
su - sonarqube 
```


```
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.6.1.59531.zip

wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.8.100196.zip
```



```
unzip sonarqube-9.9.8.100196.zip
```


```
chmod -R 755 /home/sonarqube/sonarqube-9.9.8.100196
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.9.8.100196
```



#### SonarQube Configure:


_Uncomment and modify the following lines according to your PostgreSQL setup:_

```
vim /home/sonarqube/sonarqube-9.9.8.100196/conf/sonar.properties


# DATABASE
# User credentials.
# Permissions to create tables, indices and triggers must be granted to JDBC user.
# The schema must be created first.

sonar.jdbc.username=sonar
sonar.jdbc.password=sonar123

## Add this line: 
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqubedb


# WEB SERVER
#sonar.web.host=0.0.0.0
#sonar.web.context=/sonar
#sonar.web.port=9000


# ELASTICSEARCH
# JVM options of Elasticsearch process
#sonar.search.javaOpts=-Xmx512m -Xms512m -XX:MaxDirectMemorySize=256m -XX:+HeapDumpOnOutOfMemoryError


# OTHERS

##Also add the following Elasticsearch storage paths: 
#sonar.path.data=/var/sonarqube/data
#sonar.path.temp=/var/sonarqube/temp

# LOGGING

#sonar.log.level=INFO
#sonar.path.logs=logs

```


#### For version 9.4:

_If Need:_
```
vim /home/sonarqube/sonarqube-9.9.8.100196/bin/linux-x86-64/sonar.sh

RUN_AS_USER=rcms
```


#### Start the SonarQube server:: 
Note: Can not run elasticsearch as `root`.


_Navigate to the bin folder:_

```
su - sonarqube
```


```
cd /home/sonarqube/sonarqube-9.9.8.100196/bin/linux-x86-64/
```


```
./sonar.sh -h
```


```
./sonar.sh console
```

Or,


```
./sonar.sh start
```




### Access SonarQube

By default sonar server is running on port `9000`. After sonarqube is up, provide the username and password as `admin` and `admin`. So now all done.

Access SonarQube via a web browser using `http://server_ip:9000`



### Create a project:

- Project display name* : 
- Project key* : 
- Main branch name* : 



### Token Generate: 

- Select your project "project1" - click "Locally" - click "Generate". 





---
---



## Run SonarQube Analysis

To analyze your code using SonarQube, you need the SonarScanner. Here’s how to use it:



### For a Maven Project:

```
mvn clean verify sonar:sonar \
  -Dsonar.projectKey=your_project_key \
  -Dsonar.host.url=http://your_sonar_server:9000 \
  -Dsonar.login=your_token
```


```
mvn clean verify sonar:sonar \
  -Dsonar.projectKey=projec-rcms \
  -Dsonar.host.url=http://192.168.10.190:9000 \
  -Dsonar.login=sqp_baxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```




### For a Gradle Project:

```
gradle sonarqube \
  -Dsonar.projectKey=your_project_key \
  -Dsonar.host.url=http://your_sonar_server:9000 \
  -Dsonar.login=your_token
```




### Links:

- [Sonarqube Distribution](https://binaries.sonarsource.com/?prefix=Distribution/sonarqube/)
- [Install and configure Sonarqube](https://devopscube.com/setup-and-configure-sonarqube-on-linux/)
- [Install the Sonarqube server ](https://docs.sonarsource.com/sonarqube-server/9.6/setup-and-upgrade/install-the-server/)
- [CICD pipeline with jenkins](https://medium.com/@dinukaekanayaka18/create-a-cicd-pipeline-with-jenkins-in-a-ec2-instance-to-deploy-an-application-on-to-kubernetes-89e8d3b4502a/)



Remember to consult the official SonarQube documentation for any updates or specific configurations based on your requirements.


