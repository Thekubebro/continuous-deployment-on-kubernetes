# Jenkins on Kubernetes Engine
     This topic teaches you best practices for using Jenkins with Google Kubernetes Engine. To implement this solution, see  [setting up Jenkins on Kubernetes Engine](https://cloud.google.com/solutions/jenkins-on-kubernetes-engine-tutorial) .
Jenkins is an open-source automation server that lets you flexibly orchestrate your build, test, and deployment pipelines. Kubernetes Engine is a hosted version of Kubernetes, a powerful cluster manager and orchestration system for containers.
When you need to set up a continuous delivery (CD) pipeline, deploying Jenkins on Kubernetes Engine provides important benefits over a standard VM-based deployment:
	* 	When your build process uses containers, one virtual host can run jobs against different operating systems.
	* 	Kubernetes Engine provides ephemeral build executors, allowing each build to run in a clean environment that’s identical to the builds before it.
	* 	As part of the ephemerality of the build executors, the Kubernetes Engine cluster is only utilized when builds are actively running, leaving resources available for other cluster tasks such as batch processing jobs.
	* 	Build executors launch in seconds.
	* 	Kubernetes Engine leverages the  [Google global load balancer](http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44824.pdf)  to route web traffic to your instance. The load balancer handles SSL termination, and provides a global IP address that routes users to your web front end on one of the fastest paths from the point of presence closest to your users through the Google backbone network.


## Deploying Kubernetes Engine clusters
When deploying a Kubernetes Engine cluster, enable  [authentication scopes](https://cloud.google.com/docs/authentication#oauth_scopes)  for the following services.
	* 	Google Cloud Storage — storage-rw
	* 	Jenkins needs access to Google Container Registry, a service that stores content in Cloud Storage.
	* 	Google Cloud Source Repositories — projecthosting
	* 	Cloud Source Repositories provide private repositories that you can use for your source code. Jenkins can pull from these repositories to build, test and deploy your code as quickly as possible.
For example, when creating a cluster, you could use code similar to the following:


```
gcloud container clusters create jenkins-cd \
         --network jenkins \
         --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
```
If you plan to leverage other Google services, you can add  [additional scopes](https://cloud.google.com/sdk/gcloud/reference/container/clusters/create#--scopes) .


## Deploying the Jenkins master
The following image describes the architecture for deploying Jenkins in a multi-node Kubernetes cluster.
<a href='jenkins-kubernetes-architecture.svg'>jenkins-kubernetes-architecture.svg</a>
Deploy the Jenkins master into a separate  [namespace](http://kubernetes.io/docs/admin/namespaces/)  in the Kubernetes cluster. Namespaces allow for  [creating quotas](http://kubernetes.io/docs/admin/resourcequota/)  for the Jenkins deployment as well as logically separating Jenkins from other deployments within the cluster.
To create a Kubernetes namespace, type the following command.

```
kubectl create namespace jenkins
```
## Creating Jenkins services
Jenkins provides two services that the cluster needs access to. Deploy these services separately so they can be individually managed and named.
	* 	An externally-exposed  [NodePort service](http://kubernetes.io/docs/user-guide/services/#type-nodeport)  on port 8080 that allows pods and external users to access the Jenkins user interface. This type of service can be load balanced by an HTTP load balancer.
	* 	An internal, private  [ClusterIP service](http://kubernetes.io/docs/user-guide/services/#publishing-services---service-types)  on port 50000 that the Jenkins executors use to communicate with the Jenkins master from inside the cluster.
The following sections show sample service definitions.
 [jenkins/k8s/service_jenkins.yaml](https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes/blob/master/jenkins/k8s/service_jenkins.yaml) 
 [VIEW ON GITHUB](https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes/blob/master/jenkins/k8s/service_jenkins.yaml) 


- - - -

```
  kind: Service
  apiVersion: v1
  metadata:
    name: jenkins-ui
    namespace: jenkins
  spec:
    type: NodePort
    selector:
      app: master
    ports:
      - protocol: TCP
        port: 8080
        targetPort: 8080
        name: ui
```
 [jenkins/k8s/service_jenkins.yaml](https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes/blob/master/jenkins/k8s/service_jenkins.yaml) 
 [VIEW ON GITHUB](https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes/blob/master/jenkins/k8s/service_jenkins.yaml) 


- - - -
```
  kind: Service
  apiVersion: v1
  metadata:
    name: jenkins-discovery
    namespace: jenkins
  spec:
    selector:
      app: master
    ports:
      - protocol: TCP
        port: 50000
        targetPort: 50000
        name: slaves
```
## Creating the Jenkins deployment
Deploy the Jenkins master as a  [deployment](http://kubernetes.io/docs/user-guide/deployments)  with a  [replica count](http://kubernetes.io/docs/user-guide/replication-controller/#multiple-replicas)  of 1. This ensures that there is a single Jenkins master running in the cluster at all times. If the Jenkins master pod dies or the node that it is running on shuts down, Kubernetes restarts the pod elsewhere in the cluster.
It's important to set  [requests and limits](http://kubernetes.io/docs/user-guide/compute-resources/)  as part of the deployment definition, so that the container is guaranteed a certain amount CPU and memory resources inside the cluster before being scheduled. Otherwise, your master could go down due to CPU or memory starvation.
The Jenkins home volume stores XML configuration files and plugin JAR files that make up your configuration. Consider the following best practices.
	* 	Create a persistent disk to store the home directory to ensure that your critical configuration data is maintained, even if the pod running your Jenkins master goes down.
	* 	The disk should be greater than or equal to 10 GB.
	* 	After your volume is created, you must format the volume by attaching it to a virtual machine. Then, unmount the volume from any virtual machines and detach it. You can then use the volume in your deployment.
The following code describes an example deployment definition.
 [jenkins/k8s/jenkins.yaml](https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes/blob/master/jenkins/k8s/jenkins.yaml) 
 [VIEW ON GITHUB](https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes/blob/master/jenkins/k8s/jenkins.yaml) 


```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: jenkins
  namespace: jenkins
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: master
    spec:
      containers:
      - name: master
        image: jenkins/jenkins:2.67
        ports:
        - containerPort: 8080
        - containerPort: 50000
        readinessProbe:
          httpGet:
            path: /login
            port: 8080
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 2
          failureThreshold: 5
        env:
        - name: JENKINS_OPTS
          valueFrom:
            secretKeyRef:
              name: jenkins
              key: options
        - name: JAVA_OPTS
          value: '-Xmx1400m'
        volumeMounts:
        - mountPath: /var/jenkins_home
          name: jenkins-home
        resources:
          limits:
            cpu: 500m
            memory: 1500Mi
          requests:
            cpu: 500m
            memory: 1500Mi
      volumes:
      - name: jenkins-home
        gcePersistentDisk:
          pdName: jenkins-home
          fsType: ext4
          partition: 1
```


## Connecting to Jenkins
All pods in the Kubernetes cluster can access the Jenkins user interface using master.jenkins.svc for the hostname and 8080 for the port.
Once the Jenkins pod has been created you can create a load balancer endpoint to connect to it from outside of Cloud Platform. Consider the following best practices.
	* 	Use a Kubernetes  [ingress](http://kubernetes.io/docs/user-guide/ingress/#what-is-ingress)  resource for an easy-to-configure L7 load balancer with SSL termination.
	* 	Provide SSL certs to the load balancer using Kubernetes secrets. Use tls.cert and tls.key values, and reference the values in your ingress resource configuration.


## Configuring Jenkins
### Securing Jenkins
After you connect to Jenkins for the first time, it’s important to immediately secure Jenkins. You can follow  [the Jenkins standard security setup tutorial](https://wiki.jenkins-ci.org/display/JENKINS/Standard+Security+Setup)  for a simple procedure that leverages an internal user database. This setup doesn’t require additional infrastructure and provides the ability to lock out anonymous users.
### Installing plugins
You can install the following plugins to enhance the interactions between Jenkins and Kubernetes Engine.
	* 	The  [Kubernetes plugin](https://wiki.jenkins-ci.org/display/JENKINS/Kubernetes+Plugin)  enables using Kubernetes service accounts for authentication, and creating labeled executor configurations with different base images. The plugin creates a pod when an executor is required and destroys the pod when a job ends.
	* 	The  [Google Authenticated Source plugin](https://wiki.jenkins-ci.org/display/JENKINS/Google+Source+Plugin)  enables using your service account credentials when accessing Cloud Platform services such as Cloud Source Repositories.
Install the plugins  [by following these instructions](https://jenkins.io/doc/book/managing/plugins/) .
## Configuring Jenkins for Kubernetes Engine
For step-by-step guidance on how to configure Jenkins for Kubernetes Engine, including build executor configuration, adding credentials, and more, see  [Configuring Jenkins for Kubernetes Engine](https://cloud.google.com/solutions/configuring-jenkins-kubernetes-engine) .
## Customizing the Docker image
When creating a pod template, you can either provide an existing Docker image, or you can create a custom image that has most of your build-time dependencies installed. Using a custom image can decrease overall build time and create more consistent build environments.
Your custom Docker image must install and configure the  [Jenkins JNLP slave agent](https://github.com/jenkinsci/docker-jnlp-slave) . The JNLP agent is software that communicates with the Jenkins master to coordinate running your Jenkins jobs and reporting job status.
One option is to add FROM jenkins/jnlp-slave to your image configuration. For example, if your application build process depends on the Go runtime, you can create the following Dockerfile to extend the existing image with your own dependencies and build artifacts.


```
FROM jenkins/jnlp-slave
RUN apt-get update && apt-get install -y golang
```
Then, build and upload the image to your project’s Container Registry repository by running the following commands.

```
docker build -t gcr.io/[PROJECT]/my-jenkins-image .
```

```
gcloud docker -- push gcr.io/[PROJECT]/my-jenkins-image
```
When creating a pod template, you can now set the Docker image field to the following string, where [PROJECT] is replaced with your project name and [IMAGE_NAME] is replaced with the image name.

```
gcr.io/[PROJECT]/[IMAGE_NAME]
```
The above example ensures that the Go language runtime is pre-installed when your Jenkins job starts.


## Upgrading Jenkins
To upgrade the Jenkins master,  [edit your deployment manifest](http://kubernetes.io/docs/user-guide/deployments/#updating-a-deployment)  and change the tag value for the image property to the one version you’d like to upgrade to. For an example of where the Jenkins version is defined in the manifest,  [see this manifest file](https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes/blob/master/jenkins/k8s/jenkins.yaml#L29) . For a list of available versions, see  [Docker’s Jenkins image tags list](https://hub.docker.com/r/library/jenkins/tags/) .
After you’ve edited the file, deploy the change using the following command.

```
kubectl apply -f jenkins.yaml
```
