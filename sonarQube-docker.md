
## SonarQube on Docker:

To install and run SonarQube using Docker, you can use the official SonarQube Docker image. Here's a simple guide to get you started:



### Prerequisites:
- Docker installed
- Docker Compose (optional, but recommended for PostgreSQL setup)



### Option 1: Run with Docker only (demo or quick start):

By default, the server running within the container will listen on port 9000. You can expose the container port 9000 to the host port 9000 with the `-p 9000:9000` argument to docker run, like the command below:

`Note`: By default, the image will use an **embedded H2 database** that is not suited for production.


```
docker run --name sonarqube -dit -p 9000:9000 sonarqube:community
```


#### Installing SonarScanner:

SonarScanner is a CLI tool that is used to run SonarQube tests. To view the test results, SonarScanner sends feedback from the test to the SonarQube server for you to review.

You can run SonarScanner from the Docker image with the below command:


```
docker run --rm -e SONAR_HOST_URL="<http://$>{SONARQUBE_URL}" -e SONAR_LOGIN="myAuthenticationToken" -v "${YOUR_REPO}:/usr/src" sonarsource/sonar-scanner-cli
```



### Option 2: Run with Docker Compose (recommended for persistence + PostgreSQL):

Create a `docker-compose.yml` file:

```
version: '3'

services:
  sonarqube:
    image: sonarqube:community
    container_name: sonarqube
    depends_on:
      - sonar_db
    ports:
      - "9000:9000"
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://sonar_db:5432/sonar
      - SONAR_JDBC_USERNAME=sonar
      - SONAR_JDBC_PASSWORD=sonar
    volumes:
      - ./sonarqube_data:/opt/sonarqube/data
      - ./sonarqube_extensions:/opt/sonarqube/extensions
      - ./sonarqube_logs:/opt/sonarqube/logs

  sonar_db:
    image: postgres:16
    container_name: sonar_db
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
      - POSTGRES_DB=sonar
    volumes:
      - ./sonar_pg_data:/var/lib/postgresql/data

#volumes:
#  sonarqube_data:
#  sonarqube_extensions:
#  sonarqube_logs:
#  sonar_pg_data:

```


```
docker compose -f docker-compose.yml config
```


```
docker compose up -d
```



### Access SonarQube:

Once everything is up, go to: `http://server_ip:9000`

Default credentials:
- Username: admin
- Password: admin




### Create a Local Project:

1. Click on “Create Project” and provide the project name and key.
    - Project display name* : project-jenkins
    - Project key* : project-jenkins
    - Main branch name* : main
    - click `Next`

2. Use global settings
    - [✓] Use the global setting
    - click  `Create Project`



### Token Generate::
- Select your project `project-jenkins` - click `Locally`:
    - Token name* : Analyze "project-jenkins"
    - click `Generate`.
    - click on `Continue`
    - **copy token** and Save : sqp_cec0a4ee5bebab1b693d2a1e12858cb68fc3a62e


### For Maven: 

The last step is select the build system. In my case it’s **Maven** and you could see, that SonarQube already suggests you the command, which would help you execute analysis later.

```
mvn clean verify sonar:sonar \
  -Dsonar.projectKey=projec-jenkins \
  -Dsonar.projectName='projec-jenkins' \
  -Dsonar.host.url=http://192.168.10.193:9000 \
  -Dsonar.token=sqp_cec0a4ee5bebab1b693d2a1e12858cb68fc3a62e
```



### Links:

- [SonarQube | hub.docker.com](https://hub.docker.com/_/sonarqube)
- [SonarQube | github.com](https://github.com/SonarSource/docker-sonarqube)
- [Installing SonarQube Server | docs.sonarsource.com](https://docs.sonarsource.com/sonarqube-server/latest/setup-and-upgrade/install-the-server/installing-sonarqube-from-docker/)
- [SonarQube | Docker](https://polygontechnology.io/install-sonarqube-in-locally-using-docker/)
- [SonarQube with Docker](https://medium.com/@denis.verkhovsky/sonarqube-with-docker-compose-complete-tutorial-2aaa8d0771d4)


