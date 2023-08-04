# Docker Compose
Setting up a local CI/CD-environment with Gitea, Jenkins and SonarQube.
```yaml
version: "3.1"

services:
  jenkins:
    image: jenkins/jenkins:2.416-jdk17
    ports:
      - "8080:8080"
    volumes:
      - jenkins_home:/var/jenkins_home
  ssh-agent:
    image: jenkins/ssh-agent
  sonarqube:
    image: sonarqube:10.1-community
    restart: unless-stopped
    environment:
      - SONARQUBE_JDBC_USERNAME=sonarqube
      - SONARQUBE_JDBC_PASSWORD=sonarpass
      - SONARQUBE_JDBC_URL=jdbc:postgresql://db:5432/sonarqube
    ports:
      - "9000:9000"
      - "9092:9092"
    volumes:
      - sonarqube_conf:/opt/sonarqube/conf
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_bundled-plugins:/opt/sonarqube/lib/bundled-plugins
    depends_on:
      - sonarqube-db
  sonarqube-db:
    image: postgres:14
    restart: unless_stopped
    environment:
      - POSTGRES_USER=sonarqube
      - POSTGRES_PASSWORD=sonarpass
      - POSTGRES_DB=sonarqube
    volumes:
      - sonarqube_db:/var/lib/postgresql
      - postgresql_data:/var/lib/postgresql/data
  gitea:
    image: gitea/gitea:1.20.1-rootless
    environment:
      - GITEA__database__DB_TYPE=postgres
      - GITEA__database__HOST=db:5433
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=gitea
    restart: always
    volumes:
      - gitea-data:/var/lib/gitea
      - gitea-config:/etc/gitea
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000"
      - "2222:2222"
    depends_on:
      - gitea_db
  gitea_db:
    image: postgres:14
    restart: always
    environment:
      - POSTGRES_USER=gitea
      - POSTGRES_PASSWORD=gitea
      - POSTGRES_PORT=5432
      - POSTGRES_DB=gitea
    volumes:
      - gitea-postgres-data:/var/lib/postgresql/data
volumes:
  jenkins_home:
  postgresql_data:
  sonarqube_db:
  sonarqube_conf:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_bundled-plugins:
  gitea-config:
  gitea-data:
  gitea-postgres-data:
```
