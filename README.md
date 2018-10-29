# Helm chart for deploying a generic application to Kubernetes

A helm chart which helps with deploying a generic image to Kubernetes environment 

## Usage

Consider a simple Java application for which I have a docker image available [here](https://hub.docker.com/r/bijukunjummen/sample-boot-knative-app/) - https://hub.docker.com/r/bijukunjummen/sample-boot-knative-app/. This is a simple Spring Boot applicaiton. This helm chart will help with deploying/upgrading such an application in a Kubernetes environment with the following features:

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
  repository: bijukunjummen/sample-boot-knative-app
  tag: 0.0.3-SNAPSHOT    

ingress:
  enabled: true
  path: /
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "route"

resources:
  constraints: 
    enabled: true
  requests:
    cpu: 500m
```

and the helm installed the following way:

```sh
helm repo add bk-charts http://bijukunjummen.github.io/generic-app-chart
helm repo update

# The chart should now show up in the search..
helm search generic-app-chart

# NAME                            CHART VERSION   APP VERSION     DESCRIPTION                              
# bk-charts/generic-app-chart     *****                           A generic Helm Chart for deploying an app
```


Fill in a `sample-values.yaml` file with content similar to one above.

and install the application:

```sh
helm install bk-charts/generic-app-chart  --name my-app --values sample-values.yaml
```

An output along these lines should be expected:

```
NAME:   my-app
LAST DEPLOYED: Sun Oct 28 20:45:28 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/HorizontalPodAutoscaler
NAME                      AGE
my-app-generic-app-chart  1s

==> v1/Pod(related)

NAME                                       READY  STATUS   RESTARTS  AGE
my-app-generic-app-chart-59c76c46c4-fqwvr  0/2    Pending  0         0s

==> v1/Secret

NAME                      AGE
my-app-generic-app-chart  1s

==> v1/Service
my-app-generic-app-chart  1s

==> v1/Deployment
my-app-generic-app-chart  1s

==> v1beta1/Ingress
my-app-generic-app-chart  1s


NOTES:
1. Get the application URL by running these commands:
  kubectl get --namespace default ing my-app-generic-app-chart

```


**Upgrading App**

The app can be upgraded by specifying changed values in `sample-values.yml` and running the following command:

```sh
helm upgrade my-app bk-charts/generic-app-chart -f sample-values.yaml
```