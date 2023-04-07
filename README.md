# Kubernetes demo
Deploy springboot app on a k8s cluster.

In this project, we setup Kubernetes deployment for a spring boot application. 

Additionally, we also demo Kustomize and Argocd setup.

Project for spring boot app being referred: https://github.com/ramit21/docker-demo.

We discuss 2 approaches to run the image on minikube cluster: either run all config yamls manually using kubctl,
or have Argocd listen to the Kustomize yaml of dev overlay to setup all K8s resources.

## Project setup
1. Check the docker-demo project, where the application's docker image is built and uploaded to dockerhub.
2. Setup minikube on local machine. Commands:
```
minikube start
minikube status
minikube stop
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

## Argocd setup
1. First install and setup Helm:

Run kubectl version, and notice the version mentioned against GitVersion field. Now find the compatible helm version from 
https://helm.sh/docs/topics/version_skew/, and download the compatible helm version from https://github.com/helm/helm/releases.
Unpack the tar file, and set env variable in PATH variable to the helm.exe. Verify helm installation by running 'helm list' command.

2. Install Argocd using Helm:

```
git clone https://github.com/argoproj/argo-helm.git
cd argo-helm/charts/argo-cd/
kubectl create ns myargo
helm dependency up
helm install myargo . -f values.yaml -n myargo  //(was facing some redis-ha issue, so removed the dependency form charts.yaml and it worked)
kubectl port-forward service/myargo-argocd-server 8090:80 -n myargo
http://localhost:8090 -> username is admin, password you can get from below command.
kubectl -n myargo get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d
helm list -n myargo
helm uninstall myargo -n myargo //to remove argocd
```


## Features used in this project
1. Any change to config yaml files, run 'kustomize build' on local to ensure no syntax issues with yamls.
2. We override yaml configs in the overlay folder using Kustomize features. 
 To demonstrate, we override the env variable (prop.env) for the application (initially defined in application.properties of app code) in the dev overlay config.yaml. When the application is deployed for dev env, and the app started, it will pick up the value from config file, and override with what was given in application.properties.
3. On similar lines, you can even patch the deployment yaml, giving snapshots versions (generated from feature branches) for testing in lower environments, before feature branches are merged into main.
4. Notice how we connect to dockerhub to download the image using secret of type docker-registry with the dockerhub credentials. 
5.  


