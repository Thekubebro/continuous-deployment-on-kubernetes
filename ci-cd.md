# Continuous Deployment to Kubernetes Engine using Jenkins
		
		
This tutorial shows you how to set up a continuous delivery pipeline using Jenkins and Google Kubernetes Engine, as described in the following diagram.
<a href='jenkins-cd-container-engine.svg'>jenkins-cd-container-engine.svg</a>


## Objectives
	* 	Understanding a sample application.
	* 	Deploying an application to Kubernetes.
	* 	Uploading code to Google Cloud Source Repositories.
	* 	Creating deployment pipelines in Jenkins.
	* 	Deploying development environments.
	* 	Deploying a canary release.
	* 	Deploying production environments.


## Costs
This tutorial uses billable components of Google Cloud Platform (GCP), including:
	* 	Google Compute Engine
	* 	Google Kubernetes Engine
Use the  [Pricing Calculator](https://cloud.google.com/products/calculator/#id=7ecbf600-bb93-4d6d-8381-d986f33dc9a0)  to generate a cost estimate based on your projected usage for this tutorial.


## Before you begin
	1	Select or create a GCP project.
 [GO TO THE MANAGE RESOURCES PAGE](https://console.cloud.google.com/cloud-resource-manager) 
	2	Make sure that billing is enabled for your project.
 [LEARN HOW TO ENABLE BILLING](https://cloud.google.com/billing/docs/how-to/modify-project) 
	3	Enable the Google Compute Engine, Google Kubernetes Engine APIs.  [ENABLE THE APIS](https://console.cloud.google.com/flows/enableapi?apiid=compute_component,container) 



## Preparing your environment
	1	Complete the  [Setting up Jenkins on Kubernetes Engine tutorial](https://cloud.google.com/solutions/jenkins-on-kubernetes-engine-tutorial) .
	2	In Google Cloud Shell, navigate to the sample application directory.

```
cd continuous-deployment-on-kubernetes/sample-app
```



## Understanding the application
You'll deploy the sample application, gceme, in your continuous deployment pipeline. The application is written in the Go language, and is located in the repo's sample-app directory. When you run the gceme binary on a Compute Engine instance, the app displays the instance's metadata in an info card.
![](2018-07-05/jenkins-cd-gceme-info-card.png)
The application mimics a microservice by supporting two operation modes.
	* 	In backend mode, gceme listens on port 8080 and returns Compute Engine instance metadata in JSON format.
	* 	In frontend mode, gceme queries the backend gceme service, and renders the resulting JSON in the user interface.
<a href='jenkins-cd-gceme-arch.svg'>jenkins-cd-gceme-arch.svg</a>
The frontend and backend modes support two additional URLs.
	* 	/version prints the running version.
	* 	/healthz reports the application's health. In frontend mode, the health displays as OK if the backend is reachable.


## Deploying the sample app to Kubernetes
Deploy the gceme frontend and backend to Kubernetes using manifest files that describe the deployment environment. The files use a default image that is updated later in this tutorial.
Deploy the applications into two environments.
	* 	Production. The live site that your users access.
	* 	Canary. A smaller-capacity site that receives a percentage of your user traffic. Use this environment to sanity check your software with live traffic before it's released to the live environment.
First, deploy your application into the production environment to seed the pipeline with working code.
	1	Create the Kubernetes namespace to logically isolate the production deployment.

```
kubectl create ns production
```

	2	Create the canary and production deployments and services.

```
kubectl --namespace=production apply -f k8s/production
kubectl --namespace=production apply -f k8s/canary
kubectl --namespace=production apply -f k8s/services
```

	3	Scale up the production environment frontends.

```
kubectl --namespace=production scale deployment gceme-frontend-production --replicas=4
```

	4	Retrieve the external IP for the production services. It can take several minutes before you see the load balancer IP address.

```
kubectl --namespace=production get service gceme-frontend
```

When complete, an IP address displays in the EXTERNAL-IP column. 

NAME             CLUSTER-IP      EXTERNAL-IP      PORT(S)   AGE
	5	gceme-frontend   10.79.241.131   104.196.110.46   80/TCP    5h
	6	

	7	Store the frontend service load balancer IP in an environment variable.

export FRONTEND_SERVICE_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}"  --namespace=production services gceme-frontend)

	8	Confirm that both services are working by opening the frontend external IP address in your browser.
	9	Open a separate terminal and poll the production endpoint's /version URL so you can observe rolling updates in the next section.

```
while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1;  done
```



## Creating a repository to host the sample app source code
Next, create a copy of the gceme sample app and push it to  [Cloud Source Repositories](https://cloud.google.com/source-repositories/docs/) .
	1	Create the Cloud Source Repository.

gcloud beta source repos create default

	2	Initialize the local Git repository.


```
git init
git config credential.helper gcloud.sh
```

Replace [PROJECT_ID] with your current project ID in the following command. To find your current project ID you can run gcloud config list project.

```
git remote add origin https://source.developers.google.com/p/[PROJECT_ID]/r/default
```

	3	Set the username and email address for your Git commits in this repository.

```
git config --global user.email "[EMAIL_ADDRESS]"
```

Replace [EMAIL_ADDRESS] with your Git email address.

```
git config --global user.name "[USERNAME]"
```

Replace [USERNAME] with your Git username.
	4	Add, commit, and push the files.


```
git add .
git commit -m "Initial commit"
git push origin master
```



## Creating a pipeline
Use Jenkins to define and run a pipeline for testing, building, and deploying your copy of gceme to your Kubernetes cluster.
## Adding your service account credentials
Configure your credentials to allow Jenkins to access the code repository.
	1	In the Jenkins user interface, Click Credentials in the left navigation.
	2	Click the Jenkins link in the Credentials table.
![](2018-07-05/jenkins-credential-groups.png)
	3	Click Global Credentials.
	4	Click Add Credentials in the left navigation.
	5	Select Google Service Account from metadata from the Kind dropdown.
	6	Click OK.
There are now two global credentials. Make a note of second credential's name for use later on in this tutorial.
![](2018-07-05/jenkins-cd-credentials.png)
## Creating a Jenkins job
Next, use the  [Jenkins Pipeline](https://jenkins.io/solutions/pipeline/)  feature to configure the build pipeline. Jenkins Pipeline files are written using a  [Groovy-like syntax](http://www.groovy-lang.org/) .
Navigate to your Jenkins user interface and follow these steps to configure a Pipeline job.
	1	Click the Jenkins link in the top left of the interface.
	2	Click the New Item link in the left navigation.
	3	Name the project sample-app, then choose the Multibranch Pipeline option, then click OK.
	4	On the next page, click Add Source and select git.
	5	Paste the HTTPS clone URL of your sample-app repo in Cloud Source Repositories into the Project Repository field. Replace [PROJECT_ID] with your project ID.

https://source.developers.google.com/p/[PROJECT_ID]/r/default

	6	From the Credentials dropdown, select the name of the credentials you created when adding your service account.
	7	Select the checkbox Build Periodically under Build Triggers, and enter five asterisks (* * * * *) into the Schedule field. This ensures that Jenkins checks your code repository for changes once every minute. This field uses the  [CRON expression](https://wikipedia.org/wiki/Cron#CRON_expression)  syntax to define the schedule.
	8	Click Save.
![](2018-07-05/jenkins-cd-clone-url.png)
After you complete these steps, a job named "Branch indexing" runs. This meta-job identifies the branches in your repository and ensures changes haven't occurred in existing branches. If you refresh Jenkins, the master branch displays this job.
The first run of the job fails until you make a few code changes in the next step.
## Modifying the Pipeline definition
Create a branch for the canary environment, called canary.

```
git checkout -b canary
```
The Jenkinsfile container that defines that pipeline is written using the  [Jenkins Pipeline Groovy syntax](https://jenkins.io/doc/book/pipeline/syntax/) . Using a Jenkinsfile allows an entire build pipeline to be expressed in a single file that lives alongside your source code. Pipelines support powerful features like parallelization and requiring manual user approval.
Modify the Jenkinsfile so that it contains your project ID on line 2.


## Deploying a canary release
Now that your pipeline is configured properly, it's time to make a change to the gceme application and let your pipeline test, package, and deploy it.
The canary environment is set up as a  [canary release](http://martinfowler.com/bliki/CanaryRelease.html) . As such, your change is released to a small percentage of the pods behind the production load balancer. You accomplish this in Kubernetes by maintaining multiple deployments that share the same labels. For this application, the gceme-frontend services load balance across all pods that have the labels app: gceme and role: frontend. The k8s/frontend-canary.yaml canary manifest file sets the replicas to 1 and includes labels required for the gceme-frontend service.
Currently, you have 1 out of 5 of the frontend pods running the canary code while the other 4 are running the production code. This helps ensure that the canary code doesn't negatively affect users before rolling out to your full fleet of pods.
	1	Open html.go and replace the two instances of blue with orange.
	2	Open main.go and change the version number from 1.0.0 to 2.0.0.

```
 const version string = "2.0.0"
```

	3	Next, add and commit those files to your local repository.


```
 git add Jenkinsfile html.go main.go
 git commit -m "Version 2"
```

	4	Finally, push your changes to the remote Git server.

```
 git push origin canary
```

	5	After the change is pushed to the Git repository, navigate to the Jenkins user interface where you can see that your build started.
![](2018-07-05/jenkins-cd-first-build.png)
	6	After the build is running, click the down arrow next to the build in the left navigation and select Console Output.
![](2018-07-05/jenkins-cd-console.png)
	7	Track the output of the build for a few minutes and watch for the kubectl --namespace=production apply... messages to begin. When they start, check back on the terminal that was polling the production /version URL and observe begin to change in some of the requests. You have now rolled out that change to a subset of users.
	8	After the change is deployed to the canary environment, you can continue to roll it out to the rest of your users by merging the code with the master branch and pushing that to the Git server.

```
git checkout master
git merge canary
git push origin master
```

	9	In approximately one minute, the master job in the sample-app folder kicks off.
![](2018-07-05/jenkins-cd-production.png)
	10	Clicking on the master link shows you the stages of your pipeline, as well as pass/fail information and timing characteristics.
![](2018-07-05/jenkins-cd-production-pipeline.png)
	11	Poll the production URL to verify that the new version 2.0.0 has been rolled out and is serving requests from all users.


```
export FRONTEND_SERVICE_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1;  done
```

You can look at the Jenkinsfile in the project to see the workflow.


## Deploying a development branch
Sometimes you need to work with non-trivial changes that can't be pushed directly to the canary environment. A development branch is a set of environments your developers use to test their code changes before submitting them for integration into the live site. These environments are a scaled-down version of your application, but are deployed using the same mechanisms as the live environment.
To create a development environment from a feature branch, you can push the branch to the Git server and let Jenkins deploy your environment. In a development scenario, you wouldn't use a public-facing load balancer. To help secure your application you can use  [kubectl proxy](http://kubernetes.io/docs/user-guide/connecting-to-applications-proxy/) . The proxy authenticates itself with the Kubernetes API, and proxies requests from your local machine to the service in the cluster without exposing your service to the Internet.
	1	Create another branch and push it to the Git server.


```
git checkout -b new-feature
git push origin new-feature
```

A new job is created and your development environment is in the process of being created. At the bottom of the console output of the job are instructions for accessing your environment.
	1	Start the proxy in the background.

```
kubectl proxy &
```

	2	Verify that your application is accessible by using localhost.

```
curl http://localhost:8001/api/v1/proxy/namespaces/new-feature/services/gceme-frontend:80/
```

	3	You can now push code to this branch to update your development environment. When you are done, merge your branch back into canary to deploy that code to the canary environment.

```
git checkout canary
git merge new-feature
git push canary
```

	4	When you are confident that your code won't cause problems in the production environment, merge from the canary branch to the master branch to kick off the deployment.

```
git checkout master
git merge canary
git push master
```

	5	When you are done with the development branch, delete it from the server and delete the environment from your Kubernetes cluster.

```
git push origin :new-feature
kubectl delete ns new-feature
```



## Cleaning up
To avoid incurring charges to your Google Cloud Platform account for the resources used in this tutorial:
## Deleting the project

The easiest way to eliminate billing is to delete the project you created for the tutorial.
To delete the project:
	1	Warning: Deleting a project has the following consequences:
	* 	If you used an existing project, you'll also delete any other work you've done in the project.
	* 	You can't reuse the project ID of a deleted project. If you created a custom project ID that you plan to use in the future, delete the resources inside the project instead. This step ensures that URLs that use the project ID, such as an appspot.com URL, remain available.
	2	If you are exploring multiple tutorials and quickstarts, reusing projects instead of deleting them prevents you from exceeding project quota limits. 

	3	In the GCP Console, go to the Projects page.  [GO TO THE PROJECTS PAGE](https://console.cloud.google.com/iam-admin/projects) 
	4	In the project list, select the project you want to delete and click Delete project. ![](2018-07-05/delete-project-screenshot.png)
	5	In the dialog, type the project ID, and then click Shut down to delete the project.
