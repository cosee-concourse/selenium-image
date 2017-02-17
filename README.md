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
Job description:
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
