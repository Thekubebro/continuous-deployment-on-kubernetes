# Configuring Jenkins for Kubernetes Engine
		
This page shows how to configure Jenkins for use with Google Kubernetes Engine. The tasks relate to best practices described in  [Jenkins and Kubernetes Engine](https://cloud.google.com/solutions/jenkins-on-kubernetes-engine) , including how to configure two recommended plugins, the  [Kubernetes plugin](https://wiki.jenkins-ci.org/display/JENKINS/Kubernetes+Plugin)  and  [Google Authenticated Source plugin](https://wiki.jenkins-ci.org/display/JENKINS/Google+Source+Plugin) .


## Adding credentials
Add the two sets of credentials that are used with Kubernetes Engine.
## Adding Google service account credentials
These credentials enable Jenkins to leverage your Kubernetes Engine cluster's service account to create, view, and delete cloud resources.
	1	In Jenkins, click Credentials, then click System in the left navigation.
	2	Click Global credentials (unrestricted).
	3	Click Add Credentials in the left navigation.
	4	Select Google Service Account from metadata from the Kind dropdown.
	5	Click the OK button.
## Adding Kubernetes credentials
These credentials enable the Kubernetes plugin to use the Kubernetes service account for creating, viewing, and deleting resources in the Kubernetes context.
There are two ways to add your Kubernetes credentials.
	* 	If you’ve already set up Jenkins to run inside a Kubernetes Engine or Kubernetes cluster, such as through the  [tutorial steps](https://cloud.google.com/solutions/jenkins-on-kubernetes-engine-tutorial) , you can set up a service account.
	* 	If you haven’t set up Jenkins to run inside a Kubernetes Engine or Kubernetes cluster, you must add basic HTTP authentication credentials to authenticate with the Kubernetes HTTP API.
Using a service account
	1	In Jenkins, click Credentials, then click System in the left navigation.
	2	Click Global credentials (unrestricted).
	3	Click Add Credentials in the left navigation.
	4	Select Kubernetes Service Account from the Kind dropdown. This option doesn’t appear unless Jenkins is running inside a Kubernetes Engine or Kubernetes cluster.
	5	Add a description in the Description field, such as jenkins.
	6	Click the OK button.
Using HTTP credentials
Get your Kubernetes credentials in one of two ways:
	* 	From the command line, type the following command, replacing [CLUSTER_NAME] with your cluster name and [ZONE] with your zone.

```
gcloud container clusters describe [CLUSTER_NAME] --zone [ZONE] | grep password
```

	* 	From  [Google Cloud Platform Console](https://console.cloud.google.com/) , click Kubernetes Engine from the menu, then click your cluster name. You can copy your credentials by clicking Show credentials.
Then, set up the HTTP credentials by following these steps.
	1	In Jenkins, click Credentials, then click System in the left navigation.
	2	Click Global credentials (unrestricted).
	3	Click Add Credentials in the left navigation.
	4	Select Username with password from the Kind dropdown.
	5	Enter your username and password.
	6	Click the OK button.


## Configuring the build executors
The Jenkins build executor defines the environment where your build steps run. When running Jenkins in Kubernetes Engine, the Kubernetes plugin creates and deletes lightweight executors based on an existing Docker image for each build. Jenkins provides a generic executor image, but tweaking the executors can make builds more efficient.
	1	In Jenkins, click Manage Jenkins in the left navigation. Then click Configure System.
	2	Enter 0 for # of executors. Setting this value to 0 ensures that all builds run in newly provisioned containers instead of the Jenkins master.
	3	Scroll to the bottom of the screen, then click the Add a new cloud dropdown and select Kubernetes.
	4	Enter kubernetes for the Name field.
	5	For the Kubernetes URL field:
	* 	If you’ve already set up Jenkins to run inside a Kubernetes Engine or Kubernetes cluster, Enter the following value:

```
https://kubernetes.default
```

	* 	If you provisioned your Jenkins master outside of a Kubernetes cluster, use one of the following two options to get the Kubernetes URL:
	a	Run the following command:

```
kubectl cluster-info
```

Use the full URL listed next to Kubernetes master.
	b	From  [Google Cloud Platform Console](https://console.cloud.google.com/) , click Kubernetes Engine from the menu, then click the cluster name. Use the following string for the Kubernetes URL, where [ENDPOINT] is replaced with the Endpoint value.

```
https://[ENDPOINT]
```

	6	Select the credential you set up earlier in the topic from the Credentials dropdown.
	7	For the Jenkins URL field:
	* 	If you’ve set up Jenkins to run inside a Kubernetes Engine or Kubernetes cluster, enter the following value:

```
http://jenkins-ui.jenkins.svc.cluster.local:8080
```

	* 	If not, enter the URL in the following format, replacing [JENKINS_HOST] with your Jenkins IP address or hostname:

```
http://[JENKINS_HOST]:8080
```

	8	For the Jenkins tunnel field:
	* 	If you’ve set up Jenkins to run inside a Kubernetes Engine or Kubernetes cluster, enter the following value:

```
jenkins-discovery.jenkins.svc.cluster.local:50000
```

	* 	If not, enter the URL in the following format, replacing [JENKINS_HOST] with your Jenkins IP address or hostname:

```
[JENKINS_HOST]:50000
```

![](Configuring%20Jenkins%20for%20Kubernetes%20Engine/jenkins-container-engine-kubernetes-config.png)
	9	Click the Add Pod Template dropdown, then click Kubernetes Pod Template. A pod template defines a class of executors that run your builds.
	10	Enter a name for the Name field.
	11	Enter optional values in the Labels field. The format is a space-separated list of strings. By adding labels, Jenkins jobs can use executors that are best suited for their use case. For example, you could enter the following values:

```
debian-8 docker-enabled gcloud-installed
```

	12	Enter a Docker image path for the Docker image field. The Docker image defines the environment that your build scripts run in. You can use your own image path, or enter the following path to use a prebuilt image that includes the Cloud SDK.

gcr.io/cloud-solutions-images/jenkins-k8s-slave

![](Configuring%20Jenkins%20for%20Kubernetes%20Engine/jenkins-container-engine-kubernetes-pod-config.png)
	13	As a final optional step, you can configure the pod template to allow you to use Docker from your Jenkins job definitions. This enables creating, building, running and pushing Docker images as part of your build process. To configure this option:
	a	Click the Add Volume dropdown in the Volumes field, then click Host Path Volume. This volume mounts the Docker client binary to the executor.
	b	Enter the following string for the Host path field:

```
/usr/bin/docker
```

	c	Enter the following string for the Mount path field:

```
/usr/bin/docker
```

	d	Click the Add Volume dropdown, then click Host Path Volume. This volume mounts the Docker socket so your executor can communicate with the Docker engine.
	e	Enter the following string for the Host path field:

```
/var/run/docker.sock
```

	f	Enter the following string for the Mount path field:

```
/var/run/docker.sock
```

![](Configuring%20Jenkins%20for%20Kubernetes%20Engine/jenkins-container-engine-volumes-config.png)
