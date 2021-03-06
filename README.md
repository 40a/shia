# SHIA - JUST DO IT!

# Disclaimer

Howdy partner! You stumbled upon SHIA, a CI/CD tool that aims at glueing together gitlab and rancher to orchestrate docker containers. This is a work in progress, feel free to be inspired by it and fork it. I'll be writing a series of blog posts in the next couple of weeks and continue to get SHIA in shape to be used and configured more easily.

If you stumble upon any challenges, shoot me a mail or open an issue.

![shia](https://i.ytimg.com/vi/ZXsQAXx_ao0/maxresdefault.jpg "JUST DO IT!")

# Introduction

For an introduction to SHIA, check out my slides:

https://docs.google.com/presentation/d/1FdB6myROKGYwkBoddskk7GQKblQBxPrTfBy3hA5RfiA/edit?usp=sharing

# Configuration

## repo config
We need shia config files in the repo:
* shia/compose/docker-compose.yml
* shia/compose/rancher-compose.yml
* shia/config.yml

### shia/config.yml

```yaml
---
group: testgroup
project: testproject
```

### config/
Take a look inside these files, and search for a TODO, :)
* docker_registry.yml
* git_config.yml
* google_config.yml
* stacks.yml

## gitlab-runner configuration
We need to setup our gitlab-runners:
* create a gitlab user that has a deploy key
* enable the deploy key for all projects in gitlab that are part of your stack
* have the id_rsa of that user mounted at /keys/id_rsa in the gitlab-runner docker defaults
* have the secrets.yml mounted at /keys/secrets.yml in the gitlab-runner docker defaults

### /keys/secrets.yml

```yaml
RANCHER_URL: 'https://rancher.example.com/v1'
RANCHER_ACCESS_KEY: 'secret'
RANCHER_SECRET_KEY: 'secret+1'
```

### GCE Rancher server instance
It's important, that you set the 'Cloud API access scope' of the rancher-server instance for 'Compute' to 'Read Write'. You'll not be able to add hosts with docker-machine if you forget to do it.

# Testing - recording the vcr cassettes

run:

```bash
./vcr_record.rb
```

# Usage in gitlab-ci.yml

## deploying to production environment
We're using an example project here: 'service.git' and 'production' as the production environment.

NOTE: IMAGE_NAME is now automatically generated by shia after this schema:

* docker.example.com/service:${CI_BUILD_REF}

```yaml
build:
  image: docker:latest
  stage: build
  script:
    - "docker build -t registry.example.com/group/project:${CI_BUILD_REF} ."
    - "docker push registry.example.com/group/project:${CI_BUILD_REF}"
  only:
    - master

shia_deploy:
  stage: deploy
  image: registry.example.com/deployment/shia:latest

  script:
    - shia -e production deploy
  only:
    - master
```

## setup and deploy QA environment
We're using an example project here: 'service.git' and 'feature/KCITECH-1/something/@something-env' as branch (CI_BUILD_REF_NAME).

This will create an environment:

* something-env

```yaml
deploy_qa_environment:
  stage: deploy_qa
  image: registry.example.com/deployment/shia:latest
  before_script:
    - "docker build -t registry.example.com/group/service:${CI_BUILD_REF} ."
    - "docker push registry.example.com/group/service:${CI_BUILD_REF}"
  script:
    - shia -b ${CI_BUILD_REF_NAME} deploy_all
  except:
    - master
  when: manual
```

# Testing
to run tests:
```bash
guard
```

## websockets
When the upgraded containers are done creating the service's state will change from upgrading to upgraded and the old ones are stopped. You can watch the state by polling the links.self URL of the service, or through the /v1/subscribe?eventNames=resource.change WebSocket.

# Rancher
we use rancher for our infrastructure, the installation can be found here:
https://rancher.example.com/env/1a5/apps/stacks

for infos on rancher go here:
http://rancher.com/

## API infos
to explore the api go to:
https://rancher.example.com/v1/projects/

for documentation of the api:
https://github.com/rancher/api-spec/blob/master/specification.md

the rancher-api gem is build on 'her', for infos visit:
https://github.com/remiprev/her

# Gem
## Installation
install geminabox and set gemhost to gems.example.com:
```bash
gem install geminabox
gem inabox -c
```

and install the gem:
```bash
gem install shia
```

## Usage
```bash
shia --help
```
