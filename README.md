# Wizzie training - Kubernetes CI/CD lab

 To generate this readme: `node readme.js`

## Interactive Tutorial Version
To complete the tutorial using the interactive script:

* Clone this repository.

* To run: `npm install`

Begin the desired section:

* `npm run part1`

* `npm run part2`


## Manual Tutorial Version


To complete the tutorial manually, follow the steps below.


## Part 1

#### Step1

Start up the Kubernetes cluster with Minikube, giving it some extra resources.

`minikube start --memory 8000 --cpus 2`

#### Step2

Enable the Minikube add-ons Heapster and Ingress.

`minikube addons enable heapster; minikube addons enable ingress`

#### Step3

Wait 20 seconds, and then view the Minikube Dashboard, a web UI for managing deployments.

`sleep 20; minikube service kubernetes-dashboard --namespace kube-system`

#### Step4

Deploy the public nginx image from DockerHub into a pod. Nginx is an open source web server that will automatically download from Docker Hub if it’s not available locally.

`kubectl run nginx --image nginx --port 80`

#### Step5

Create a service for deployment. This will expose the nginx pod so you can access it with a web browser.

`kubectl expose deployment nginx --type NodePort --port 80`

#### Step6

Launch a web browser to test the service. The nginx welcome page displays, which means the service is up and running.

`minikube service nginx`

#### Step7

Set up the cluster registry by applying a .yml manifest file.

`kubectl apply -f manifests/registry.yml`

#### Step8

Wait for the registry to finish deploying. Note that this may take several minutes.

`kubectl rollout status deployments/registry`

#### Step9

View the registry user interface in a web browser.

`minikube service registry-ui`

#### Step10

Let’s make a change to an HTML file in the cloned project. Open the /applications/hello-wizzie/index.html file in your favorite text editor (for example, you can use nano by running the command 'nano applications/hello-wizzie/index.html' in a separate terminal). Change some text inside one of the <p> tags. For example, change “Hello from Wizzie!” to “Hello from Me!”. Save the file.

`nano applications/hello-wizzie/index.html`

#### Step11

Now let’s build an image, giving it a special name that points to our local cluster registry.

`docker build -t 127.0.0.1:30400/hello-wizzie:latest -f applications/hello-wizzie/Dockerfile applications/hello-wizzie`

#### Step12

We’ve built the image, but before we can push it to the registry, we need to set up a temporary proxy. By default the Docker client can only push to HTTP (not HTTPS) via localhost. To work around this, we’ll set up a container that listens on 127.0.0.1:30400 and forwards to our cluster.

`docker stop socat-registry; docker rm socat-registry; docker run -d -e "REGIP=`minikube ip`" --name socat-registry -p 30400:5000 chadmoon/socat:latest bash -c "socat TCP4-LISTEN:5000,fork,reuseaddr TCP4:`minikube ip`:30400"`

#### Step13

With our proxy container up and running, we can now push our image to the local repository.

`docker push 127.0.0.1:30400/hello-wizzie:latest`

#### Step14

The proxy’s work is done, so you can go ahead and stop it.

`docker stop socat-registry;`

#### Step15

With the image in our cluster registry, the last thing to do is apply the manifest to create and deploy the hello-wizzie pod based on the image.

`kubectl apply -f applications/hello-wizzie/k8s/deployment.yaml`

#### Step16

Launch a web browser and view the service.

`minikube service hello-wizzie`

## Part 2

#### Step1

Deploy the ClusterRoleBinding located in manifests/rbac.yaml

`kubectl apply -f manifests/rbac.yaml`

#### Step2

Install Jenkins, which we’ll use to create our automated CI/CD pipeline. It will take the pod a minute or two to roll out.

`kubectl apply -f manifests/jenkins.yml; kubectl rollout status deployment/jenkins`

#### Step3

Open the Jenkins UI in a web browser.

`minikube service jenkins`

#### Step4

Display the Jenkins admin password with the following command, and right-click to copy it. IMPORTANT: BE CAREFUL NOT TO PRESS CTRL-C TO COPY THE PASSWORD AS THIS WILL STOP THE SCRIPT.

`kubectl exec -it `kubectl get pods --selector=app=jenkins --output=jsonpath={.items..metadata.name}` cat /root/.jenkins/secrets/initialAdminPassword`

#### Step5

Switch back to the Jenkins UI. Paste the Jenkins admin password in the box and click Continue. Click Install suggested plugins and wait for the process to complete.

`echo ''`

#### Step6

Create an admin user and credentials, and click Save and Finish. (Make sure to remember these credentials as you will need them for repeated logins.) Click Start using Jenkins.

`echo ''`

#### Step7

We now want to create a new pipeline for use with our Hello-Wizzie app. On the left, click New Item. Enter the item name as "Hello-Wizzie Pipeline", select Pipeline, and click OK.

`echo ''`

#### Step8

Under the Pipeline section at the bottom, change the Definition to be "Pipeline script from SCM".

`echo ''`

#### Step9

Change the SCM to Git.

`echo ''`

#### Step10

Change the Repository URL to be the URL of your forked Git repository, such as https://github.com/[GIT USERNAME]/kubernetes-ci-cd. Click Save. On the left, click Build Now to run the new pipeline.

`echo ''`

#### Step11

Now view the Hello-Wizzie application.

`minikube service hello-wizzie`

#### Step12

Push a change to your fork. Run job again. View the changes.

`minikube service hello-wizzie`