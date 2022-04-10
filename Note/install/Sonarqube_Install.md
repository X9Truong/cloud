### SonarQube install on Centos 7

* If you're running on Linux, you must ensure that:

```
* vm.max_map_count is greater than or equal to 524288
* fs.file-max is greater than or equal to 131072
* the user running SonarQube can open at least 131072 file descriptors
* the user running SonarQube can open at least 8192 threads
```

* If the user running SonarQube (sonarqube in this example) does not have the permission to have at least 131072 open descriptors, you must insert this line in /etc/security/limits.d/99-sonarqube.conf (or /etc/security/limits.conf as you wish):

```
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
```

```
sysctl -w vm.max_map_count=262144
sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```
* Start a basic container:

`docker run -d --name sonarqube -p 9000:9000 sonarqube`

* Login
```
http://192.168.30.69:9000/
User name : admin
Password: admin
```

* SonarQube container with Persistent volume attached:

`docker run -d -p 9000:9000 -v sonarqube_conf:/opt/sonarqube/conf -v sonarqube_extensions:/opt/sonarqube/extensions -v sonarqube_logs:/opt/sonarqube/logs -v sonarqube_data:/opt/sonarqube/data sonarqube`


## SonarQube with Postgresql DockerCompose

```
vim docker-compose.yml
version: "3.3"
services:
  db:
    image: postgres:12-alpine
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
      - POSTGRES_DB=sonar
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - sonarqube-net

  sonarqube:
    image: sonarqube:community
    depends_on:
      - db
    environment:
      - sonar.jdbc.username=sonar
      - sonar.jdbc.url=jdbc:postgresql://db/sonar
      - sonar.jdbc.password=sonar
    ports:
      - 9000:9000
    volumes:
      - sonar_conf:/opt/sonarqube/conf
      - sonar_data:/opt/sonarqube/data
      - sonar_extensions:/opt/sonarqube/extensions
      - sonar_plugins:/opt/sonarqube/lib/bundled-plugins
    networks:
      - sonarqube-net
networks:
  sonarqube-net:
volumes:
  sonar_conf:
  sonar_data:
  sonar_extensions:
  sonar_plugins:
  postgres_data:
```

Refer: 


```
https://docs.sonarqube.org/latest/analysis/analysis-parameters/
https://docs.sonarqube.org/latest/requirements/requirements/
```
