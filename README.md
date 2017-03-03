# Selenium WebDriver Docker Image [![Docker Repository on Quay](https://quay.io/repository/cosee-concourse/selenium/status "Docker Repository on Quay")](https://quay.io/repository/cosee-concourse/selenium)

This Docker image provides support for Selenium WebDriver Tests.

This image includes following technologies:
- Maven
- Firefox (headless)
- Chromium (headless)

If you want to use Selenium Standalone, feel free to use [cosee-concourse/dind-image](https://github.com/cosee-concourse/dind-image) instead.

This image is hosted at quay.io: [quay.io/cosee-concourse/selenium](https://quay.io/cosee-concourse/selenium)
## Example: Usage with ConcourseCI
### Selenium WebDriver
Task definition:
``` yaml
platform: linux

image_resource:
        type: docker-image
        source: {
        repository: quay.io/cosee-concourse/selenium,
        tag: "latest" }

run:
        path: sh
        args:
        - -exc
        - |
          Xvfb :98 &
          export DISPLAY=:98
          cd source/path_to_selenium-tests
          #example maven selenium test
          mvn -Dweb.baseUrl=https://$APPURL/ install
          kill -9 $(pgrep Xvfb)

inputs:
    - name: source

```
There are different tags of this image. (For more details, take a look on quay.io).


### Selenium WebDriver using Docker-Compose
If you want to use Selenium WebDriver with Docker-Compose, you have to use the DinD image (Further information on [cosee-concourse/dind-image](https://github.com/cosee-concourse/dind-image).

Pipeline definition:
```yaml
...

jobs:
- name: Test
  plan:
  - get: source
    trigger: true
  - task: runTest
    privileged: true        # required
    file: source/path_to_task_definition/taskDefinition.yml
    params:
        APPNAME: testingappname
        CREDENTIAL1: cred1
        CREDENTIAL2: cred2
    
... 
```

Task definition:
``` yaml
platform: linux

image_resource:
        type: docker-image
        source: {
        repository: quay.io/cosee-concourse/dind,
        tag: "latest" }

run:
        path: sh
        args:
        - -exc
        - |
          source /docker-lib.sh         #required
          start_docker                  #required
          export BASEFOLDER=$(pwd)      #recommended

          #set up test environment
          docker-compose -f source/path_to_dockercompose/dc-testenvironment.yml up -d
          #Selenium Test
          docker-compose -f source/path_to_dockercompose/dc-testscenario.yml run selenium
          rc=$?
          #Test 2
          docker-compose -f source/path_to_dockercompose/dc-testscenario.yml run test2
          rc2=$?
          #destroy test environment
          docker-compose -f source/path_to_dockercompose/dc-testenvironment.yml down
          docker-compose -f source/path_to_dockercompose/dc-testscenario.yml down
          exit $rc || $rc2

inputs:
    - name: source
    - name: artifacts
```

dc-testenvironment.yml:
``` yaml
version: '3'
services:
    mysqlserver:
        image: mysql:5.6
        container_name: "mysqlserver"
        ports:
            - "3306:3306"
        volumes:
            - ./tmp/mysql:/var/lib/mysql
        restart: always
        environment:
            - MYSQL_ROOT_PASSWORD=ThisPassword
        networks:
            - selenium-network
    frontend:
        image: "maven:3-jdk-8"
        container_name: "frontend"
        links: 
            - "mysqlserver"
        networks:
            - selenium-network
        ports:
            - "81:81"
        volumes:
            - ${BASEFOLDER}/artifacts:/artifacts
        environment: 
            - NAME=${APPNAME}
            - CRED1=${CREDENTIAL1}
            - CRED2=${CREDENTIAL2}
        command: >
            /bin/sh -c "
                while ! curl mysqlserver:3306 2> /dev/null;
                do
                    echo \"Waiting for MySQL server\";
                    sleep 1;
                done;
                echo \"Server is available!\";

                cd /artifacts
                java \
                    -Dserver.port=81 \
                    -jar testapp-0.0.1-SNAPSHOT.jar -Xmx300m -Xms50m \
                    --spring.datasource.url=jdbc:mysql://mysqlserver/testdatabase"

networks:
    selenium-network:
        driver: bridge
```

dc-testscenario.yml:
``` yaml
version: '3'
services:
    selenium:
        image: "quay.io/cosee-concourse/selenium"
        container_name: "selenium"
        network_mode: "host"
        volumes:
            - ${BASEFOLDER}/source:/source
            - /dev/shm:/dev/shm             #required for Chromium Driver
        environment:
            - NAME=${APPNAME}
            - CRED1=${CREDENTIAL1}
        command: >
            /bin/sh -c "
                Xvfb :98 &
                export DISPLAY=:98
                while ! timeout 1 bash -c 'cat < /dev/null > /dev/tcp/localhost/81';
                do
                    echo \"Waiting for website\";
                    sleep 1;
                done;
                echo \"Server is available!\";
                cd source/path_to_selenium-tests
                #example maven selenium test
                mvn -Dweb.baseUrl=https://$APPURL/ install
                kill -9 $(pgrep Xvfb)"
                
    test2:
        image: "maven"
        container_name: "test2"
        network_mode: "host"
        ...
    
    ...
 
 
```

### Selenium Standalone
If you want to use Selenium Standalone, you have to use the DinD image (Further information on [cosee-concourse/dind-image](https://github.com/cosee-concourse/dind-image).

docker-compose.yml:
``` yaml
hub:
  image: selenium/hub
  ports:
    - "4444:4444"
firefox:
  image: selenium/node-firefox
  links:
    - hub
chrome:
  image: selenium/node-chrome
  links:
    - hub
```

Pipeline definition:
```yaml
...

jobs:
- name: Selenium-Test
  plan:
  - get: source
    trigger: true
  - task: runSeleniumTest
    privileged: true        # required
    file: source/path_to_task_definition/taskDefinition.yml
    
... 
```

Task definition:
``` yaml
platform: linux

image_resource:
        type: docker-image
        source: {
        repository: quay.io/cosee-concourse/dind,
        tag: "latest" }

run:
        path: sh
        args:
        - -exc
        - |
          source /docker-lib.sh               # required
          start_docker                        # required
          cd source/path_to_dockercompose_yml
          docker-compose up
          # if you want to use more instances of firefox and chrome:
          # docker-compose scale chrome=15 firefox=15
          docker run -it --rm myTestDocker /runTest.sh -p 4444
          rc=$?                               # exit code of myTestDocker
          docker-compose down                 # required
          exit $rc

inputs:
    - name: source
```
