# Kubernetes demo
Deploy spring-boot app on a k8s cluster.

In this project, we setup Kubernetes deployment for a spring boot application. 

Additionally, we also demo Kustomize and Argocd setups.

Project for spring boot app being referred: https://github.com/ramit21/docker-demo.

We discuss 2 approaches to run the image on minikube cluster: either run all config yamls manually using kubctl,
or have Argocd listen to the Kustomize yaml of dev overlay to setup all K8s resources.

## Project setup
1. Check the docker demo project on my github repos, where the application's docker image is built and uploaded to docker hub.
2. Set up minikube on the local machine. Commands:
```
minikube start
minikube status
minikube stop
minikube delete
```
3. Create a docker secret by running this command, replace PASSWORD with the docker account's password (as also used in the docker-demo project when uploading the image using the DMP plugin to the docker hub account)
```
kubectl create secret docker-registry dockerhub-secret --docker-server=docker.io --docker-username=ramit21 --docker-password=PASSWORD --docker-email=21.ramit@gmail.com
```
4. Install Kustomize from https://github.com/kubernetes-sigs/kustomize/releases
5. To create k8s resources, go to the k8-config folder of the project and do kubectl apply -f <file.yaml> for each resource config, or do it via argocd setup as explained below.
6. Once all resources including the node port service have been created, run the below command to get URL to access the app locally. Once you open it, you will notice the application has picked up overridden env variable from config.yaml.
ie. the app in the browser says 'Hello World from env = dev' instead of env = local.
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

3. cd into argocd folder of the project, and create the argocd application. Open Argocd UI, and you will see resources provisioned (screenshot added at root for reference)
```
kubectl apply -f application.yaml
```

4. As done in step 6 above, and call the application from the browser. Note that this time, the service name is prefixed with dev-, and the namespace is demo-app.
```
minikube service dev-mydeployment --url -n demo-app
```

## Features used in this project
1. For Any change to config yaml files, run 'kustomize build' on local to ensure no syntax issues with yamls.
2. We override yaml configs in the overlay folder using Kustomize features. 
 To demonstrate, we override the env variable (prop.env) for the application (initially defined in application.properties of app code) in the dev overlay config.yaml. When the application is deployed for dev env, and the app started, it will pick up the value from config file, and override with what was given in application.properties.
3. On similar lines, you can even patch the deployment yaml, giving snapshots versions (generated from feature branches) for testing in lower environments, before feature branches are merged into main.
4. Notice how we connect to dockerhub to download the image using secret of type docker-registry with the dockerhub credentials. 
5. Theory on ArgoCd is covered here: https://github.com/ramit21/argo-cd
6. Things to notice in overlays/dev/Kustomization.yaml: we give a namespace at one place, and all resources get created in it. 
   Note how it refers to resources in the base folder, and then applies **patches** to update the base resource with a modified version of the spec.
   You can also use **patchesJson6902** to add more elements to the base resource yamls, eg add more Istio virtual service routes.
7. argocd/application.yaml: note that we give the same namespace, where argocd was setup initially. 
	Also, note how we give prefixes and labels that get applied to all k8s resources created.
	Note how the path is pointing to the dev overlay. So argocd running on the dev cluster will point to dev overlay, and likewise for prod.
	Any commit to source k8-configs (including app image version), argocd will pick up and apply on the cluster.
8. Argocd UI: you can click on the pod, and check application logs.
   When syncing - select prune to remove resources that have been renamed or removed from k8s-configs.
   When syncing - select Force, if you are pushing changes to the application against the same image version, and want to pick up the latest. The deployment should also have imagePullAlways set as true for this to work.   

