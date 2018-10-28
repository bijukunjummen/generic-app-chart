# Helm chart for deploying a generic application to Kubernetes

A helm chart which helps with deploying a generic image to Kubernetes environment 

## Usage

Consider a simple Java application for which I have a docker image available [here](https://hub.docker.com/r/bijukunjummen/sample-backend-app/) - https://hub.docker.com/r/bijukunjummen/sample-backend-app/. This is a simple Spring Boot applicaiton that exposes its endpoint at port 8082 by default. This helm chart will help with deploying/upgrading such an application in a Kubernetes environment with the following features:

1. It installs a deployment to manage the set of pods with a readiness probe and liveness probes 
1. It installs a HPA to manage the scaling of pods based on load
1. It uses a Kubernetes secrets to store the secrets associated with the application
1. It creates an ingress with a configurable set of annotations

The override values look like this :

```yaml
app:
  healthCheckPath: /actuator/info
  environment:
    SERVER_PORT: "8080"
    ENV_NAME: ENV_VALUE
  secrets:
    SECRET1: SECRET_VALUE1
  autoscaling:
    enabled: true
    maxReplicas: 2
    minReplicas: 1
    targetCPUUtilizationPercentage: 40    


image:
  repository: bijukunjummen/sample-backend-app
  tag: 0.0.3-SNAPSHOT    

ingress:
  enabled: true
  path: /
```