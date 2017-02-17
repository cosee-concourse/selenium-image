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
Job definition:
``` yaml
platform: linux

image_resource:
        type: docker-image
        source: {
        repository: quay.io/cosee-concourse/selenium,
        tag: "latest"}

run:
        path: sh
        args:
        - -exc
        - |
          Xvfb :98 &
          export DISPLAY=:98
          cd source/path_to_selenium-tests
          #example maven selenium test
          mvn -Dweb.baseUrl=https://$APPURL/ clean install
          kill -9 $(pgrep Xvfb)

inputs:
    - name: source

```
There are different tags of this image. (For more details, take a look on quay.io).

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
    file: source/path_to_job_definition/jobDefinition.yml
    
... 
```

Job definition:
``` yaml
platform: linux

image_resource:
        type: docker-image
        source: {
        repository: quay.io/cosee-concourse/dind,
        tag: "latest",
        privileged: true }

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
