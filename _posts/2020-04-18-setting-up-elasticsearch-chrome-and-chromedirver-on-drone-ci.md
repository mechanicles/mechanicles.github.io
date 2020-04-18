---
layout: post
title: "Setting up Elasticsearch, Chrome, and Chromedriver on Drone CI"
tags:
- CI
- elasticsearch
- chromedriver
- rspec
- rails
- TIL
---

Our search specs are now dependent on Elasticsearch and to run
feature/system specs, we need Chrome and Chromedriver software packages in
our system. Recently, I was working on setting up Elasticsearch, Chrome, and
Chromedriver software packages on [Drone CI](https://drone.io/). 

Let's discuss how to set up those software packages and make them ready to serve.

Initially, I went through Drone docs and was able to set up Elasticsearch
quickly using this
[example](https://docker-runner.docs.drone.io/examples/service/elasticsearch/).
But I was not able to find useful information for installing Chrome and
Chromedriver software packages. Tried some docker images, but not able to make
them work.

After investigating, I manually installed Chrome and Chromedriver packages using
commands. Please check following Drone CI code `.drone.yml` and this might help
you. 

```yaml
---
kind: pipeline
name: your-app-name

platform:
  os: linux
  arch: amd64

services:
  - name: database
    image: mdillon/postgis:10
    environment:
      POSTGRES_DB: test_database
      POSTGRES_USER: postgres
    ports:
      - 5432

  - name: elasticsearch
    image: elasticsearch:5-alpine
    ports:
      - 9200

steps:
  - name: test
    image: alpine:3.8
    commands:
      - apk add curl
      - sleep 20
      - curl http://elasticsearch:9200

  - name: tests
    image: ruby:2.5.3
    environment:
      RAILS_ENV: test
      DOCKER_CI: true
      ELASTICSEARCH_URL: http://elasticsearch:9200 # Use this in your app for setting Elasticsearch configuration using Chewy/Searchkick.
    commands:
      - apt-get update && apt-get install apt-transport-https
      - curl -sL https://deb.nodesource.com/setup_13.x | bash -
      - apt-get update && apt-get install -y nodejs
      - apt-get install -y libnss3
      - apt install -y postgresql-client
      - gem install bundler --conservative
      - bundle check || bundle install --jobs 20 --retry 5
      - cp config/drone.database.yml config/database.yml # Based on your system, change/set this.
      - wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
      - apt-get install -y ./google-chrome-stable_current_amd64.deb
      - mkdir /chromedriver
      - wget -q --continue -P /chromedriver "https://chromedriver.storage.googleapis.com/76.0.3809.126/chromedriver_linux64.zip"
      - unzip /chromedriver/chromedriver* -d /chromedriver
      - PATH=/chromedriver:$PATH
      - chromedriver -v
      - RAILS_ENV=test bundle exec rake db:create && bundle exec rake db:structure:load
      - RAILS_ENV=test bundle exec rspec spec

trigger:
  event:
    - pull_request
```

After these changes, our CI build passed successfully! 🚀

Found that chrome installation takes a bit more time. But not sure if Chrome
docker image solves this issue, if it does, I will recommend using that one.

It would be good if anyone knows useful references for installing these software
packages using docker images and making them work, please do let me know. 

Happy **CI**-ing!
