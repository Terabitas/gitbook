# Roadmap

# v1.0.0

## `api-server`

* Production ready

## `nildev io,r`

* stabilize API
* optimize generated code (optimize templates)
* At least 80% of unit tests coverage

## SPA

* Apply same principles for producing SPA docker images
* Default `spa-host` server
* Default `spa-builder`
* Easy setup of required API services for SPA development

## Deployment to kubernetes

* Apply same principles to deployment process. Absorb all complexity of dealing with kubernetes and expose higher level interface for developers. Ideally only "Quality Of Service" requirements should be defined by developer.
* Expose deployment commands through `nildev` cli app