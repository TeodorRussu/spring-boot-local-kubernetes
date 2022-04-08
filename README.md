****Simple example of dockerizing a Spring Boot app - pushing it to docker hub - and running it on local Kubernetes cluster****

**Steps to perform:**

**Prerequisites**
install docker
install kubectl: https://kubernetes.io/docs/tasks/tools/
install minikube: https://minikube.sigs.k8s.io/docs/start/


build the application:
`./mvnw install
`


**Docker tasks:**
build the docker image
`./mvnw spring-boot:build-image
`

run the container locally(optional):

`docker run -p 8080:8080 spring-boot-on-kubernetes:0.0.1
`

push the image (need to authenticate with in local running Docker application):

`docker tag spring-boot-on-kubernetes:0.0.1 teodorrussu/spring-boot-on-kubernetes`

`docker push teodorrussu/spring-boot-on-kubernetes`




**Kubernetes tasks:**

First check your running kubernetes cluster:

`kubectl cluster-info
`


We have a container that runs and exposes port 8080, so all you need to make Kubernetes run it is a deployment YAML configuration. 
kubectl can generate it for you. The only thing that might vary here is the --image name.

`kubectl create deployment spring-boot --image=teodorrussu/spring-boot-on-kubernetes --dry-run=client -o=yaml > deployment.yaml
`

`echo --- >> deployment.yaml
`

`kubectl create service clusterip spring-boot --tcp=8080:8080 --dry-run=client -o=yaml >> deployment.yaml
`


Apply the deployment:

`kubectl apply -f deployment.yaml
`

create an SSH tunnel to the application, which you have exposed as a Service in Kubernetes to:

`kubectl port-forward svc/spring-boot 8080:8080`


Now the app is up and running, and can be accessed via:

`curl localhost:8080/actuator/health
`


Delete the service from kubernetes:

`kubectl delete -n default service spring-boot`