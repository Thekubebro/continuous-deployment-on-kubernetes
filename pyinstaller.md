# Build a Python app with PyInstaller

This tutorial shows you how to use Jenkins to orchestrate building a simple Python application with  [PyInstaller](http://www.pyinstaller.org/) .
If you are a Python developer who is new to CI/CD concepts, or you might be familiar with these concepts but don’t know how to implement building your application using Jenkins, then this tutorial is for you.
The simple Python application (which you’ll obtain from a sample repository on GitHub) is a command line tool "add2vals" that outputs the addition of two values. If at least one of the values is a string, "add2vals" treats both values as a string and instead concatenates the values. The "add2" function in the "calc" library (which "add2vals" imports) is accompanied by a set of unit tests. These are tested with pytest to check that this function works as expected and the results are saved to a JUnit XML report.
The delivery of the "add2vals" tool through PyInstaller converts this tool into a standalone executable file for Linux, which you can download through Jenkins and execute at the command line on Linux machines without Python.
**Note:** Unlike the  [other tutorials](https://jenkins.io/doc/tutorials/)  in this documentation, this tutorial requires approximately 500 MB more Docker image data to be downloaded.
**Duration:** This tutorial takes 20-40 minutes to complete (assuming you’ve already met the prerequisites below). The exact duration will depend on the speed of your machine and whether or not you’ve already run Jenkins in Docker from  [another tutorial](https://jenkins.io/doc/tutorials/) .
You can stop this tutorial at any point in time and continue from where you left off.
If you’ve already run though  [another tutorial](https://jenkins.io/doc/tutorials/) , you can skip the Prerequisites and Run Jenkins in Docker sections below and proceed on to forking the sample repository. (Just ensure you have  [Git](https://git-scm.com/downloads)  installed locally.) If you need to restart Jenkins, simply follow the restart instructions in Stopping and restarting Jenkins and then proceed on.
# Prerequisites
For this tutorial, you will require:
	* 	A macOS, Linux or Windows machine with:
	◦	256 MB of RAM, although more than 512MB is recommended.
	◦	10 GB of drive space for Jenkins and your Docker images and containers.
	* 	
	* 	The following software installed:
	◦	 [Docker](https://www.docker.com/)  - Read more about installing Docker in the  [Installing Docker](https://jenkins.io/doc/book/installing/#installing-docker)  section of the  [Installing Jenkins](https://jenkins.io/doc/book/installing/)  page.
**Note:** If you use Linux, this tutorial assumes that you are not running Docker commands as the root user, but instead with a single user account that also has access to the other tools used throughout this tutorial.
	◦	 [Git](https://git-scm.com/downloads)  and optionally  [GitHub Desktop](https://desktop.github.com/) 
	* 	
# Run Jenkins in Docker
In this tutorial, you’ll be running Jenkins as a Docker container from the  [jenkinsci/blueocean](https://hub.docker.com/r/jenkinsci/blueocean/)  Docker image.
To run Jenkins in Docker, follow the relevant instructions below for either macOS and Linux or Windows.
You can read more about Docker container and image concepts in the  [Docker](https://jenkins.io/doc/book/installing#docker)  and  [Downloading and running Jenkins in Docker](https://jenkins.io/doc/book/installing#downloading-and-running-jenkins-in-docker)  sections of the  [Installing Jenkins](https://jenkins.io/doc/book/installing)  page.
# On macOS and Linux
	1	Open up a terminal window.
	2	Run the jenkinsci/blueocean image as a container in Docker using the following  [docker run](https://docs.docker.com/engine/reference/commandline/run/)  command (bearing in mind that this command automatically downloads the image if this hasn’t been done):
docker run \
	3	  --rm \
	4	  -u root \
	5	  -p 8080:8080 \
	6	  -v jenkins-data:_var_jenkins_home \ 
	7	  -v _var_run_docker.sock:_var_run_docker.sock \
	8	  -v "$HOME":/home \ 
	9	  jenkinsci/blueocean


	10	
	11	Maps the _var_jenkins_home directory in the container to the Docker  [volume](https://docs.docker.com/engine/admin/volumes/volumes/)  with the name jenkins-data. If this volume does not exist, then this docker run command will automatically create the volume for you.
	12	
	13	Maps the $HOME directory on the host (i.e. your local) machine (usually the _Users_<your-username> directory) to the /home directory in the container.

	14	
**Note:** If copying and pasting the command snippet above doesn’t work, try copying and pasting this annotation-free version here:

docker run \
	15	  --rm \
	16	  -u root \
	17	  -p 8080:8080 \
	18	  -v jenkins-data:_var_jenkins_home \
	19	  -v _var_run_docker.sock:_var_run_docker.sock \
	20	  -v "$HOME":/home \
	21	  jenkinsci/blueocean


	22	Proceed to the Setup wizard.
# On Windows
	1	Open up a command prompt window.
	2	Run the jenkinsci/blueocean image as a container in Docker using the following  [docker run](https://docs.docker.com/engine/reference/commandline/run/)  command (bearing in mind that this command automatically downloads the image if this hasn’t been done):
docker run ^
	3	  --rm ^
	4	  -u root ^
	5	  -p 8080:8080 ^
	6	  -v jenkins-data:_var_jenkins_home ^
	7	  -v _var_run_docker.sock:_var_run_docker.sock ^
	8	  -v "%HOMEPATH%":/home ^
	9	  jenkinsci/blueocean


For an explanation of these options, refer to the macOS and Linux instructions above.

	10	Proceed to the Setup wizard.
# Accessing the Jenkins/Blue Ocean Docker container
If you have some experience with Docker and you wish or need to access the Jenkins_Blue Ocean Docker container through a terminal_command prompt using the  [docker exec](https://docs.docker.com/engine/reference/commandline/exec/)  command, you can add an option like --name jenkins-tutorials (with the  [docker run](https://docs.docker.com/engine/reference/commandline/run/)  above), which would give the Jenkins/Blue Ocean Docker container the name "jenkins-tutorials".
This means you could access the Jenkins_Blue Ocean container (through a separate terminal_command prompt window) with a docker exec command like:
docker exec -it jenkins-tutorials bash
# Setup wizard
Before you can access Jenkins, there are a few quick "one-off" steps you’ll need to perform.
## Unlocking Jenkins
When you first access a new Jenkins instance, you are asked to unlock it using an automatically-generated password.
	1	After the 2 sets of asterisks appear in the terminal/command prompt window, browse to http://localhost:8080 and wait until the **Unlock Jenkins** page appears.
![](Build%20a%20Python%20app%20with%20PyInstaller/setup-jenkins-01-unlock-jenkins-page.jpg)

	2	From your terminal/command prompt window again, copy the automatically-generated alphanumeric password (between the 2 sets of asterisks).
![](Build%20a%20Python%20app%20with%20PyInstaller/setup-jenkins-02-copying-initial-admin-password.png)

	3	On the **Unlock Jenkins** page, paste this password into the **Administrator password** field and click **Continue**.
## Customizing Jenkins with plugins
After unlocking Jenkins, the **Customize Jenkins** page appears.
On this page, click **Install suggested plugins**.
The setup wizard shows the progression of Jenkins being configured and the suggested plugins being installed. This process may take a few minutes.
## Creating the first administrator user
Finally, Jenkins asks you to create your first administrator user.
	1	When the **Create First Admin User** page appears, specify your details in the respective fields and click **Save and Finish**.
	2	When the **Jenkins is ready** page appears, click **Start using Jenkins**.
**Notes:**
	◦	This page may indicate **Jenkins is almost ready!** instead and if so, click **Restart**.
	◦	If the page doesn’t automatically refresh after a minute, use your web browser to refresh the page manually.
	3	
	4	If required, log in to Jenkins with the credentials of the user you just created and you’re ready to start using Jenkins!
# Stopping and restarting Jenkins
Throughout the remainder of this tutorial, you can stop the Jenkins_Blue Ocean Docker container by typing Ctrl-C in the terminal_command prompt window from which you ran the docker run ... command above.
To restart the Jenkins/Blue Ocean Docker container:
	1	Run the same docker run ... command you ran for macOS, Linux or Windows above.
**Note:** This process also updates the jenkinsci/blueocean Docker image, if an updated one is available.
	2	Browse to http://localhost:8080.
	3	Wait until the log in page appears and log in.
# Fork and clone the sample repository on GitHub
Obtain the simple "add" Python application from GitHub, by forking the sample repository of the application’s source code into your own GitHub account and then cloning this fork locally.
	1	Ensure you are signed in to your GitHub account. If you don’t yet have a GitHub account, sign up for a free one on the  [GitHub website](https://github.com/) .
	2	Fork the  [simple-python-pyinstaller-app](https://github.com/jenkins-docs/simple-python-pyinstaller-app)  on GitHub into your local GitHub account. If you need help with this process, refer to the  [Fork A Repo](https://help.github.com/articles/fork-a-repo/)  documentation on the GitHub website for more information.
	3	Clone your forked simple-python-pyinstaller-app repository (on GitHub) locally to your machine. To begin this process, do either of the following (where <your-username> is the name of your user account on your operating system):
	◦	If you have the GitHub Desktop app installed on your machine:
	a	In GitHub, click the green **Clone or download** button on your forked repository, then **Open in Desktop**.
	b	In GitHub Desktop, before clicking **Clone** on the **Clone a Repository** dialog box, ensure **Local Path** for:
	* 	macOS is _Users_<your-username>_Documents_GitHub/simple-python-pyinstaller-app
	* 	Linux is _home_<your-username>_GitHub_simple-python-pyinstaller-app
	* 	Windows is C:\Users\<your-username>\Documents\GitHub\simple-python-pyinstaller-app
	c	
	◦	
	◦	Otherwise:
	a	Open up a terminal/command line prompt and cd to the appropriate directory on:
	* 	macOS - _Users_<your-username>_Documents_GitHub/
	* 	Linux - _home_<your-username>_GitHub_
	* 	Windows - C:\Users\<your-username>\Documents\GitHub\ (although use a Git bash command line window as opposed to the usual Microsoft command prompt)
	b	
	c	Run the following command to continue/complete cloning your forked repo:
git clone https://github.com/YOUR-GITHUB-ACCOUNT-NAME/simple-python-pyinstaller-app
where YOUR-GITHUB-ACCOUNT-NAME is the name of your GitHub account.
	◦	
	4	
# Create your Pipeline project in Jenkins
	1	Go back to Jenkins, log in again if necessary and click **create new jobs** under **Welcome to Jenkins!**
**Note:** If you don’t see this, click **New Item** at the top left.
	2	In the **Enter an item name** field, specify the name for your new Pipeline project (e.g. simple-python-pyinstaller-app).
	3	Scroll down and click **Pipeline**, then click **OK** at the end of the page.
	4	( _Optional_ ) On the next page, specify a brief description for your Pipeline in the **Description** field (e.g. An entry-level Pipeline demonstrating how to use Jenkins to build a simple Python application with PyInstaller.)
	5	Click the **Pipeline** tab at the top of the page to scroll down to the **Pipeline** section.
	6	From the **Definition** field, choose the **Pipeline script from SCM** option. This option instructs Jenkins to obtain your Pipeline from Source Control Management (SCM), which will be your locally cloned Git repository.
	7	From the **SCM** field, choose **Git**.
	8	In the **Repository URL** field, specify the directory path of your locally cloned repository above, which is from your user account/home directory on your host machine, mapped to the /home directory of the Jenkins container - i.e.
	◦	For macOS - _home_Documents_GitHub_simple-python-pyinstaller-app
	◦	For Linux - _home_GitHub/simple-python-pyinstaller-app
	◦	For Windows - _home_Documents_GitHub_simple-python-pyinstaller-app
	9	
	10	Click **Save** to save your new Pipeline project. You’re now ready to begin creating your Jenkinsfile, which you’ll be checking into your locally cloned Git repository.
# Create your initial Pipeline as a Jenkinsfile
You’re now ready to create your Pipeline that will automate building your Python application with PyInstaller in Jenkins. Your Pipeline will be created as a Jenkinsfile, which will be committed to your locally cloned Git repository (simple-python-pyinstaller-app).
This is the foundation of "Pipeline-as-Code", which treats the continuous delivery pipeline a part of the application to be versioned and reviewed like any other code. Read more about Pipeline and what a Jenkinsfile is in the  [Pipeline](https://jenkins.io/doc/book/pipeline)  and  [Using a Jenkinsfile](https://jenkins.io/doc/book/pipeline/jenkinsfile)  sections of the User Handbook.
First, create an initial Pipeline with a "Build" stage that executes the first part of the entire production process for your application. This "Build" stage downloads a Python Docker image and runs it as a Docker container, which in turn compiles your simple Python application into byte code.
	1	Using your favorite text editor or IDE, create and save new text file with the name Jenkinsfile at the root of your local simple-python-pyinstaller-app Git repository.
	2	Copy the following Declarative Pipeline code and paste it into your empty Jenkinsfile:
pipeline {
	3	    agent none 
	4	    stages {
	5	        stage('Build') { 
	6	            agent {
	7	                docker {
	8	                    image 'python:2-alpine' 
	9	                }
	10	            }
	11	            steps {
	12	                sh 'python -m py_compile sources_add2vals.py sources_calc.py' 
	13	            }
	14	        }
	15	    }
	16	}


	17	
	18	The  [agent](https://jenkins.io/doc/book/pipeline/syntax#agent)  section with the none parameter specified at the top of this Pipeline code block means that no global agent will be allocated for the entire Pipeline’s execution and that each  [stage](https://jenkins.io/doc/book/pipeline/syntax/#stage)  directive must specify its own agent section.
	19	
	20	Defines a  [stage](https://jenkins.io/doc/book/pipeline/syntax/#stage)  (directive) called Build that appears on the Jenkins UI.
	21	
	22	This image parameter (of the  [agent](https://jenkins.io/doc/book/pipeline/syntax#agent)  section’s docker parameter) downloads the  [python:2-alpine Docker image](https://hub.docker.com/_/python/)  (if it’s not already available on your machine) and runs this image as a separate container. This means that:
	◦	You’ll have separate Jenkins and Python containers running locally in Docker.
	◦	The Python container becomes the  [agent](https://jenkins.io/doc/book/glossary/#agent)  that Jenkins uses to run the Build stage of your Pipeline project. However, this container is short-lived - its lifespan is only that of the duration of your Build stage’s execution.
	23	
	24	
	25	This  [sh](https://jenkins.io/doc/pipeline/steps/workflow-durable-task-step/#code-sh-code-shell-script)  step (of the  [steps](https://jenkins.io/doc/book/pipeline/syntax/#steps)  section) runs the Python command to compile your application and its calc library into byte code files (each with .pyc extension), which are placed into the sources workspace directory (within the _var_jenkins_home_workspace_simple-python-pyinstaller-app directory in the Jenkins container).
	26	
	27	Save your edited Jenkinsfile and commit it to your local simple-python-pyinstaller-app Git repository. E.g. Within the simple-python-pyinstaller-app directory, run the commands:
git add .
then
git commit -m "Add initial Jenkinsfile"
	28	Go back to Jenkins again, log in again if necessary and click **Open Blue Ocean** on the left to access Jenkins’s Blue Ocean interface.
	29	In the **This job has not been run** message box, click **Run**, then quickly click the **OPEN** link which appears briefly at the lower-right to see Jenkins running your Pipeline project. If you weren’t able to click the **OPEN** link, click the row on the main Blue Ocean interface to access this feature.
**Note:** You may need to wait a few minutes for this first run to complete. After making a clone of your local simple-python-pyinstaller-app Git repository itself, Jenkins:
	a	Initially queues the project to be run on the agent.
	b	Downloads the Python Docker image and runs it in a container on Docker.
![](Build%20a%20Python%20app%20with%20PyInstaller/python-pyinstaller-01-downloading-python-docker-image.png)

	c	Runs the Build stage (defined in the Jenkinsfile) on the Python container. During this time, Python uses the py_compile module to compile the code of your Python application and its calc library into byte code, which are stored in the sources workspace directory (within the Jenkins home directory).
	30	
The Blue Ocean interface turns green if Jenkins compiled your Python application successfully.

![](Build%20a%20Python%20app%20with%20PyInstaller/python-pyinstaller-02-initial-pipeline-runs-successfully-step-opened.png)

	31	Click the **X** at the top-right to return to the main Blue Ocean interface.
![](Build%20a%20Python%20app%20with%20PyInstaller/python-pyinstaller-03-main-blue-ocean-interface.png)

# Add a test stage to your Pipeline
	1	Go back to your text editor/IDE and ensure your Jenkinsfile is open.
	2	Copy and paste the following Declarative Pipeline syntax immediately under the Build stage of your Jenkinsfile:
        stage('Test') {
	3	            agent {
	4	                docker {
	5	                    image 'qnib/pytest'
	6	                }
	7	            }
	8	            steps {
	9	                sh 'py.test --verbose --junit-xml test-reports_results.xml sources_test_calc.py'
	10	            }
	11	            post {
	12	                always {
	13	                    junit 'test-reports/results.xml'
	14	                }
	15	            }
	16	        }


so that you end up with:

pipeline {
	17	    agent none
	18	    stages {
	19	        stage('Build') {
	20	            agent {
	21	                docker {
	22	                    image 'python:2-alpine'
	23	                }
	24	            }
	25	            steps {
	26	                sh 'python -m py_compile sources_add2vals.py sources_calc.py'
	27	            }
	28	        }
	29	        stage('Test') { 
	30	            agent {
	31	                docker {
	32	                    image 'qnib/pytest' 
	33	                }
	34	            }
	35	            steps {
	36	                sh 'py.test --verbose --junit-xml test-reports_results.xml sources_test_calc.py' 
	37	            }
	38	            post {
	39	                always {
	40	                    junit 'test-reports/results.xml' 
	41	                }
	42	            }
	43	        }
	44	    }
	45	}


	46	
	47	Defines a  [stage](https://jenkins.io/doc/book/pipeline/syntax/#stage)  (directive) called Test that appears on the Jenkins UI.
	48	
	49	This image parameter (of the  [agent](https://jenkins.io/doc/book/pipeline/syntax#agent)  section’s docker parameter) downloads the  [qnib:pytest Docker image](https://hub.docker.com/r/qnib/pytest/)  (if it’s not already available on your machine) and runs this image as a separate container. This means that:
	◦	You’ll have separate Jenkins and pytest containers running locally in Docker.
	◦	The pytest container becomes the  [agent](https://jenkins.io/doc/book/glossary/#agent)  that Jenkins uses to run the Test stage of your Pipeline project. This container’s lifespan lasts the duration of your Test stage’s execution.
	50	
	51	
	52	This  [sh](https://jenkins.io/doc/pipeline/steps/workflow-durable-task-step/#code-sh-code-shell-script)  step (of the  [steps](https://jenkins.io/doc/book/pipeline/syntax/#steps)  section) executes pytest’s py.test command on sources/test_calc.py, which runs a set of unit tests (defined in test_calc.py) on the "calc" library’s add2 function (used by your simple Python application add2vals). The:
	◦	--verbose option makes py.test generate verbose output in the Jenkins/Blue Ocean interface.
	◦	--junit-xml test-reports_results.xml option makes py.test generate a JUnit XML report, which is saved to test-reports_results.xml (within the _var_jenkins_home_workspace_simple-python-pyinstaller-app directory in the Jenkins container).
	53	
	54	
	55	This  [junit](https://jenkins.io/doc/pipeline/steps/junit/#code-junit-code-archive-junit-formatted-test-results)  step (provided by the  [JUnit Plugin](https://jenkins.io/doc/pipeline/steps/junit) ) archives the JUnit XML report (generated by the py.test command above) and exposes the results through the Jenkins interface. In Blue Ocean, the results are accessible through the **Tests** page of a Pipeline run. The  [post](https://jenkins.io/doc/book/pipeline/syntax/#post)  section’s always condition that contains this junit step ensures that the step is _always_ executed _at the completion_ of the Test stage, regardless of the stage’s outcome.
	56	
	57	Save your edited Jenkinsfile and commit it to your local simple-python-pyinstaller-app Git repository. E.g. Within the simple-python-pyinstaller-app directory, run the commands:
git stage .
then
git commit -m "Add 'Test' stage"
	58	Go back to Jenkins again, log in again if necessary and ensure you’ve accessed Jenkins’s Blue Ocean interface.
	59	Click **Run** at the top left, then quickly click the **OPEN** link which appears briefly at the lower-right to see Jenkins running your amended Pipeline project. If you weren’t able to click the **OPEN** link, click the _top_ row on the Blue Ocean interface to access this feature.
**Note:** It may take a few minutes for the qnib:pytest Docker image to download (if this hasn’t already been done).
If your amended Pipeline ran successfully, here’s what the Blue Ocean interface should look like. Notice the additional "Test" stage. You can click on the previous "Build" stage circle to access the output from that stage.
![](Build%20a%20Python%20app%20with%20PyInstaller/python-pyinstaller-11-test-stage-runs-successfully-with-output.png)

	60	Click the **X** at the top-right to return to the main Blue Ocean interface.
# Add a final deliver stage to your Pipeline
	1	Go back to your text editor/IDE and ensure your Jenkinsfile is open.
	2	Copy and paste the following Declarative Pipeline syntax immediately under the Test stage of your Jenkinsfile:
        stage('Deliver') {
	3	            agent {
	4	                docker {
	5	                    image 'cdrx/pyinstaller-linux:python2'
	6	                }
	7	            }
	8	            steps {
	9	                sh 'pyinstaller --onefile sources/add2vals.py'
	10	            }
	11	            post {
	12	                success {
	13	                    archiveArtifacts 'dist/add2vals'
	14	                }
	15	            }
	16	        }


so that you end up with:

pipeline {
	17	    agent none
	18	    stages {
	19	        stage('Build') {
	20	            agent {
	21	                docker {
	22	                    image 'python:2-alpine'
	23	                }
	24	            }
	25	            steps {
	26	                sh 'python -m py_compile sources_add2vals.py sources_calc.py'
	27	            }
	28	        }
	29	        stage('Test') {
	30	            agent {
	31	                docker {
	32	                    image 'qnib/pytest'
	33	                }
	34	            }
	35	            steps {
	36	                sh 'py.test --verbose --junit-xml test-reports_results.xml sources_test_calc.py'
	37	            }
	38	            post {
	39	                always {
	40	                    junit 'test-reports/results.xml'
	41	                }
	42	            }
	43	        }
	44	        stage('Deliver') { 
	45	            agent {
	46	                docker {
	47	                    image 'cdrx/pyinstaller-linux:python2' 
	48	                }
	49	            }
	50	            steps {
	51	                sh 'pyinstaller --onefile sources/add2vals.py' 
	52	            }
	53	            post {
	54	                success {
	55	                    archiveArtifacts 'dist/add2vals' 
	56	                }
	57	            }
	58	        }
	59	    }
	60	}


	61	
	62	Defines a  [stage](https://jenkins.io/doc/book/pipeline/syntax/#stage)  (directive) called Deliver that appears on the Jenkins UI.
	63	
	64	This image parameter (of the  [agent](https://jenkins.io/doc/book/pipeline/syntax#agent)  section’s docker parameter) downloads the  [cdrx/pyinstaller-linux Docker image](https://hub.docker.com/r/cdrx/pyinstaller-linux/)  (if it’s not already available on your machine) and runs this image as a separate container. This means that:
	◦	You’ll have separate Jenkins and PyInstaller (for Linux) containers running locally in Docker.
	◦	The PyInstaller container becomes the  [agent](https://jenkins.io/doc/book/glossary/#agent)  that Jenkins uses to run the Deliver stage of your Pipeline project. This container’s lifespan lasts the duration of your Deliver stage’s execution.
	65	
	66	
	67	This  [sh](https://jenkins.io/doc/pipeline/steps/workflow-durable-task-step/#code-sh-code-shell-script)  step (of the  [steps](https://jenkins.io/doc/book/pipeline/syntax/#steps)  section) executes the pyinstaller command (in the PyInstaller container) on your simple Python application. This bundles your add2vals.py Python application into a single standalone executable file (via the --onefile option) and outputs the this file to the dist workspace directory (within the Jenkins home directory). Although this step consists of a single command, as a general principle, it’s a good idea to keep your Pipeline code (i.e. the Jenkinsfile) as tidy as possible and place more complex build steps (particularly for stages consisting of 2 or more steps) into separate shell script files like the deliver.sh file. This ultimately makes maintaining your Pipeline code easier, especially if your Pipeline gains more complexity.
	68	
	69	This  [archiveArtifacts](https://jenkins.io/doc/pipeline/steps/workflow-basic-steps/#code-archive-code-archive-artifacts)  step (provided as part of Jenkins core) archives the standalone executable file (generated by the pyinstaller command above at dist/add2vals within the Jenkins home’s workspace directory) and exposes this file through the Jenkins interface. In Blue Ocean, archived artifacts like these are accessible through the **Artifacts** page of a Pipeline run. The  [post](https://jenkins.io/doc/book/pipeline/syntax/#post)  section’s success condition that contains this archiveArtifacts step ensures that the step is executed /at the completion/ of the Deliver stage _only if_ this stage completed successfully.
	70	
	71	Save your edited Jenkinsfile and commit it to your local simple-python-pyinstaller-app Git repository. E.g. Within the simple-python-pyinstaller-app directory, run the commands:
git stage .
then
git commit -m "Add 'Deliver' stage"
	72	Go back to Jenkins again, log in again if necessary and ensure you’ve accessed Jenkins’s Blue Ocean interface.
	73	Click **Run** at the top left, then quickly click the **OPEN** link which appears briefly at the lower-right to see Jenkins running your amended Pipeline project. If you weren’t able to click the **OPEN** link, click the _top_ row on the Blue Ocean interface to access this feature.
**Note:** It may take a few minutes for the cdrx/pyinstaller-linux Docker image to download (if this hasn’t already been done).
If your amended Pipeline ran successfully, here’s what the Blue Ocean interface should look like. Notice the additional "Deliver" stage. Click on the previous "Test" and "Build" stage circles to access the outputs from those stages.
![](Build%20a%20Python%20app%20with%20PyInstaller/python-pyinstaller-21-deliver-stage-runs-successfully.png)

Here’s what the output of the "Deliver" stage should look like, showing you the results of PyInstaller bundling your Python application into a single standalone executable file.

![](Build%20a%20Python%20app%20with%20PyInstaller/python-pyinstaller-22-deliver-stage-output-only.png)

	74	Click the **X** at the top-right to return to the main Blue Ocean interface, which lists your previous Pipeline runs in reverse chronological order.
![](Build%20a%20Python%20app%20with%20PyInstaller/python-pyinstaller-23-main-blue-ocean-interface-with-all-previous-runs-displayed.png)

# Follow up (optional)
If you use Linux, you can try running the standalone add2vals application you generated with PyInstaller locally on your machine. To do this:
	1	From the main Blue Ocean interface, access your last Pipeline run you performed above. To do this, click the top row (representing the most recent Pipeline run) on the main Blue Ocean’s **Activity** page.
![](Build%20a%20Python%20app%20with%20PyInstaller/python-pyinstaller-23-main-blue-ocean-interface-with-all-previous-runs-displayed.png)

	2	On the results page of the Pipeline run, click **Artifacts** at the top right to access the **Artifacts** page.
![](Build%20a%20Python%20app%20with%20PyInstaller/python-pyinstaller-21-deliver-stage-runs-successfully.png)

	3	In the list of artifacts, click the down-arrow icon at the far right of the **dist/add2vals** artifact item to download the standalone executable file to your browser’s "Downloads" directory.
![](Build%20a%20Python%20app%20with%20PyInstaller/python-pyinstaller-24-deliver-stage-artifacts-page.png)

	4	Back in your operating system’s terminal prompt, cd to your browser’s "Downloads" directory.
	5	Make the add2vals file executable - i.e. chmod a+x add2vals
	6	Run the command ./add2vals and follow the instructions provided by your app.
# Wrapping up
Well done! You’ve just used Jenkins to build a simple Python application!
The "Build", "Test" and "Deliver" stages you created above are the basis for building more complex Python applications in Jenkins, as well as Python applications that integrate with other technology stacks.
Because Jenkins is extremely extensible, it can be modified and configured to handle practically any aspect of build orchestration and automation.
To learn more about what Jenkins can do, check out:
	* 	The  [Tutorials overview](https://jenkins.io/doc/tutorials)  page for other introductory tutorials.
	* 	The  [User Handbook](https://jenkins.io/doc/book)  for more detailed information about using Jenkins, such as  [Pipelines](https://jenkins.io/doc/book/pipeline)  (in particular  [Pipeline syntax](https://jenkins.io/doc/book/pipeline/syntax) ) and the  [Blue Ocean](https://jenkins.io/doc/book/blueocean)  interface.
	* 	The  [Jenkins blog](https://jenkins.io/node)  for the latest events, other tutorials and updates.
