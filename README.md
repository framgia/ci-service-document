# Framgia CI Service Document

The Vietnamese version of this document can be found [here](./README.vn.md)

## Framgia CI Service
**Framgia CI Service** contains 4 main services. They are:
- **Framgia Drone Service**: It is an open-source for running CI/CD service, written in Golang, powered by Docker.
Framgia Drone is using Drone version **0.4.2**. More documentation about Drone can be found [here](http://readme.drone.io/0.4/)
- **Framgia CI Report Service**: It is a service built by Framgia, used for storing and displaying test reports. It also helps users and administrators to track the build status more easily.
- **Framgia CI CLI**: It is a Command Line Interface written in python, used for calling Framgia CI Report Service's APIs. It is also used for running build's commands more easily and flexibly. For the installation and usage of Framgia CI CLI, you can refer its own repository https://github.com/framgia/ci-report-tool/
- **Framgia GitHub Comment Service**: It is the service that analyzes the reports and sends comments to Github. It only works with Framgia CI Report Service.
(Of course, you have to store the reports, to be able to send comments to Github :smile:)

## Framgia Drone Service
- URL: http://ci.framgia.vn/
- Requirement: You have to contact the Administrators, then they will create an account for you in http://ci.framgia.vn/ . Moreover, you must have the **administration** role in the **Repository** to be able to activate it in Drone Service.
- Framgia Drone Service can run independently with the other 3 parts. However, by only using the Drone Service, you can only check the build status in Github. If you want to store and check the test reports of each build, you have to use the other tools of **Framgia CI Service**
- After activate the project in Framgia Drone, you have to create a configuration file, to tell Drone what it has to do to make the build. This file is named `.drone.yml`, written in `yaml` syntax. You can find more information in Drone's  [document](http://readme.drone.io/0.4).<br>
An example of configuration file for Rails project
```
build:
  image: framgia/ruby-workspace-ci
  commands:
    - curl -o /usr/bin/framgia-ci https://raw.githubusercontent.com/framgia/ci-report-tool/master/dist/framgia-ci && chmod +x /usr/bin/framgia-ci
    - bundle install --path=/drone/.bundle
    - framgia-ci run
cache:
  mount:
    - .git
    - /drone/.bundle
```

As you can see, the project above will be built inside the Docker container that is created from `framgia/ruby-workspace-ci` docker image. The contains of this image you can check in **Docker Hub** with the URL https://hub.docker.com/r/framgia/ruby-workspace-ci/

About the contain of the configuration file, there are 2 parts: `build` part and `cache` part. The role of `build` part is downloading the `framgia-ci` tool, install necessary Ruby gems from the cached folder. After that, `framgia-ci` is used to run the tests, as well as send the reports.

At the end, `.git` and `/drone/.bundle` are cached, so that the the later builds can use.

- Framgia Drone has many special **plugins** (which are not officially provided by Drone) written by Framgia Members. They are used for auto deployment (with many different tools supported like **capistrano**, **rocketeer**, **ansible** ...), as well as Chatwork notification. All Drone Plugins can be found at Framgia CI Team's Docker Hub: https://hub.docker.com/r/fdplugins/

- **Notices**:
    - You should create your own Docker Image for your project, with all packages and softwares installed, so the build will become faster (during the build, you do not have to download and install other tools, packages ...)
    - You should use the cache feature, to cache some folder like `bundle`, `npm`, `composer`, so the build will become faster
    - At the current version of Framgia Drone, a Pull Request can only restore cache from the **default branch**. It can be fixed when Framgia CI Services are updated to newer version.

- You can refer the sample configurations for PHP project [here](./php/), for Ruby project [here](./ruby), for Android project [here](./android).

## Framgia CI Report Service
- URL: http://ci-reports.framgia.vn/
- Requirement: You must have a activated repository in Framgia Drone, to be able to check it in Framgia CI Report.
- Currently, Framgia CI Report Service supports 3 types of project: Ruby, PHP and Android, with various of tools supported:
    - Ruby: `bundle-audit`, `rspec` with code coverage, `brakeman`, `reek`, `rubocop`, `rails_best_practices`
    - PHP: `phpcpd` (PHP Copy/Paste Detector), `phpmd` (PHP Mess Detector), `pdepend` (PHP Depend), `phpmetrics`, `phpcs` (PHP CodeSniffer), `phpunit` with Code Coverage
    - Android: `android-lint`, `checkstyle`, `findbugs`, `pmd`)
    - Javascript & CSS (`eslint`, `scss-lint`) for PHP, Ruby, Java project
- Framgia CI Report also have Chatwork or Email notification feature. You must have the administration role in Github Repository to change these settings in Framgia CI Report Service.
- To receive notifications in **Chatwork**, you can use your own Chatwork Bot, or use the default bot that Framgia CI provides. First, add contact with [Framgia CI Bot](https://www.chatwork.com/framgia-ci-bot), then add it to the Chatwork box where you want to receive messages. Framgia CI Services will take care of other tasks for you.

## Framgia CI CLI tool
- URL: https://github.com/framgia/ci-report-tool/
- If you're using MacOS, you must have python 3.5 to install and use `framgia-ci` tool. If you're using Linux, you can install by using `pip` like Mac, or you can just download the pre-compiled file to use.
- You have to create a separate configuration file for Framgia CI CLI. It is named `.framgia-ci.yml`. This file can be auto generated by using `framgia-ci init {project_type}` command. For example:
```
framgia-ci init php
framgia-ci init ruby
framgia-ci init android
```
- The reports file which are generated during the build **must** be put inside a folder named `.framgia-ci-reports`, so that the Framgia CI Report Service can identify them.

## Framgia GitHub Comment service
- The Framgia GitHub Comment service will be run automatically after Framgia CI Report Service finishes copying reports file from Framgia Drone. You have nothing to do with this service.
