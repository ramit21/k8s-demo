# Kustomize-argocd
Deploy springboot app on a k8s cluster.

In this project, we setup Kubernetes deployment for a spring boot application. 

Additionally, we also demo Kustomize and Argocd setup.

Project for spring boot app being referred: https://github.com/ramit21/docker-demo.

## Project setup
1. Check the docker-demo project, where the application's docker image is built and uploaded to dockerhub.
2. Setup minikube on local machine. Commands:
```
minikube start
minikube stop
minikube status
minikube delete
```
3. Create docker secret by running this command, replace PASSWORD with the docker account's password (as also used in docker-demo project when uploading the image using DMP plugin to docker hub account)
```
kubectl create secret docker-registry dockerhub-secret --docker-server=docker.io --docker-username=ramit21 --docker-password=PASSWORD --docker-email=21.ramit@gmail.com
```
4. Install Kustomize from https://github.com/kubernetes-sigs/kustomize/releases
5. To create k8s resources, go to k8-config folder of the project and do kubectl apply -f <file.yaml> for each resource config, or do it via argocd setup as explained below.
6. Once all resources including the nodeport service have been created, run below command to get url to access app on local. Once you open it, you will notice the application has picked up overriden env variable from config.yaml.
ie. the app in browser says 'Hello World from env = dev' instead of env = local.
```
minikube service mydeployment --url
```

##Argocd setup
Coming soon...

## Features used in this project
1. Any change to config yaml files, run 'kustomize build' on local to ensure no syntax issues with yamls.
2. We override yaml configs in the overlay folder using Kustomize features. 
 To demonstrate, we override the env variable (prop.env) for the application (initially defined in application.properties of app code) in the dev overlay config.yaml. When the application is deployed for dev env, and the app started, it will pick up the value from config file, and override with what was given in application.properties.
3. On similar lines, you can even patch the deployment yaml, giving snapshots versions (generated from feature branches) for testing in lower environments, before feature branches are merged into main.
4. Notice how we connect to dockerhub to download the image using secret of type docker-registry with the dockerhub credentials. 
5.  


