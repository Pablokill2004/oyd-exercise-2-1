# **Exercise 2.1 — Kubernetes Manifests for a Web Application**

**Course:** Optimizaciones y Desempeño — Cloud Deployment Automation  
**Session:** 2 — April 30, 2026  
**Time allowed:** 30 minutes  
**Submission:** Initialize a new repository called ***oyd-exercise-2-1*** and commit/push everything into it.  **Submit the repository URL only.**

# Context

You will containerize a small Node.js web application and write Kubernetes manifests to deploy it to a local cluster. The application reads three environment variables and displays them in the browser.

The following files are provided — copy them exactly into your repository:

### app.js

**const** express \= require('express');  
**const** app \= express();  
**const** port \= 3000;  
*app*.get('/', (*req*, *res*) **\=\>** {  
 *res*.send(\`  
   \<h1\>Session Demo\</h1\>  
   \<p\>\<b\>APP\_ENV:\</b\> ${*process*.*env*.APP\_ENV || 'not set'}\</p\>  
   \<p\>\<b\>APP\_NAME:\</b\> ${*process*.*env*.APP\_NAME || 'not set'}\</p\>  
   \<p\>\<b\>LOG\_LEVEL:\</b\> ${*process*.*env*.LOG\_LEVEL || 'not set'}\</p\>  
 \`);  
});  
*app*.listen(*port*, () **\=\>** *console*.log(\`Listening on port ${*port*}\`));

### 

### package.json

{  
 "name": "session-demo",  
 "version": "1.0.0",  
 "main": "app.js",  
 "dependencies": {  
   "express": "^4.18.2"  
 }  
}

### Dockerfile

FROM node:20-alpine  
WORKDIR /app  
COPY package.json .  
RUN npm install \--production  
COPY app.js .  
EXPOSE 3000  
CMD \["node", "[app.js](http://app.js)"\]

 

The application listens on port 3000 and must receive these environment variables:

| Variable | Value |
| ----- | ----- |
| APP\_ENV | development |
| APP\_NAME | session-demo |
| LOG\_LEVEL | info |

# Setup

* Docker installed and running locally  
* A local Kubernetes cluster: [Docker Desktop](https://www.docker.com/products/docker-desktop/) with Kubernetes enabled, or [minikube](https://minikube.sigs.k8s.io/docs/start).  
* [kubectl](https://kubernetes.io/docs/tasks/tools/) configured against the local cluster  
* If using minikube, run *eval $(minikube docker-env)* before building the image so the image is available inside the cluster  
* Copy the three starter files into the root of your repository before starting

# Tasks

## Task 1 — Build the image

1. Build the Docker image: docker build \-t session-demo:1.0 .  
2. Confirm the image appears in docker images.

## Task 2 — Namespace

1. Write a Namespace manifest at k8s/namespace.yaml for a namespace called webapp.

## Task 3 — ConfigMap

1. Write a ConfigMap manifest at k8s/configmap.yaml named webapp-config in the webapp namespace.  
2. Include all three environment variables as key-value pairs.

## Task 4 — Deployment

1. Write a Deployment manifest at k8s/deployment.yaml named webapp in the webapp namespace with exactly 2 replicas.  
2. Use image session-demo:1.0. Set imagePullPolicy: Never so Kubernetes uses the locally built image.  
3. Inject all three values from webapp-config using envFrom referencing the ConfigMap.  
4. Set resource requests of 64m CPU and 128Mi memory, and limits of 250m CPU and 256Mi memory.  
5. Label pods with app: webapp.

## Task 5 — Service

1.  Write a ClusterIP Service manifest at k8s/service.yaml named webapp-svc in the webapp namespace.  
2. Select pods with label app: webapp.  
3. Expose port 8080 on the Service, forwarding to container port 3000.

## Task 6 — Validate and apply

1. Run kubectl apply \-f k8s/ \--dry-run=client and confirm it exits with no errors.  
2. Apply the manifests to the cluster: kubectl apply \-f k8s/  
3. Confirm both pods are running: kubectl get pods \-n webapp

## Task 7 — Port-forward and capture evidence

1. Forward the Service port to your local machine: kubectl port-forward svc/webapp-svc 8080:8080 \-n webapp  
2. Open http://localhost:8080 in your browser and verify all three environment variable values appear on the page.  
3. Take a screenshot of the browser window showing the page at http://localhost:8080. Save it as evidence/k8s-run.png in your repository.

## Task 8 — README

1. Create a README.md at the repository root.  
2. Include a \#\# Evidence section containing the screenshot rendered inline: \!\[K8s run\](evidence/k8s-run.png)  
3. Include a \#\# Validate and apply section with the exact commands from Task 6\.

# Acceptance Criteria

* Repository contains app.js, package.json, Dockerfile, all four manifests under k8s/, evidence/k8s-run.png, and README.md  
* Screenshot shows the browser at http://localhost:8080 with all three environment variable values visible — proving the app is running inside Kubernetes via port-forward  
*  README.md renders the screenshot inline under the Evidence section  
* No manifest targets the default namespace  
* kubectl apply \-f k8s/ \--dry-run=client exits with no errors  
* Deployment has 2 replicas, image session-demo:1.0, and imagePullPolicy: Never  
* All three environment variables are sourced from the ConfigMap via envFrom  
* Resource requests and limits are defined  
*  Service selector matches pod labels; Service port 8080 forwards to container port 3000