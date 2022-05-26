# EXPERIENCE CONTINUOUS INTEGRATION WITH JENKINS | ANSIBLE | ARTIFACTORY | SONARQUBE | PHP


Part 1 – Configuring Ansible For Jenkins Deployment

In previous projects, you have been launching Ansible commands manually from a CLI. Now, with Jenkins, we will start running Ansible from Jenkins UI.

Launch an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Proj14-Jenkins"

Install JDK (since Jenkins is a Java-based application)
    
    sudo apt update -y
    sudo apt install default-jdk-headless -y

Install Jenkins

    wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
        /etc/apt/sources.list.d/jenkins.list'
    sudo apt update -y
    sudo apt-get install jenkins -y

Make sure Jenkins is up and running

    sudo systemctl status jenkins

![1 d](https://user-images.githubusercontent.com/10243139/137590173-c640787a-088d-47e6-a3f3-0e4ef9abe94f.png)

Perform initial Jenkins setup.

From your browser access, navigate to:

    http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080

You will be prompted to provide a default admin password

Retrieve the default admin password from your server:

    sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Navigate to Jenkins URL

    http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080

![1 g](https://user-images.githubusercontent.com/10243139/137590326-f4d9afc5-a050-4e71-aea9-52ebe4d27f3b.png)

- Install & Open Blue Ocean Jenkins Plugin

![1 h](https://user-images.githubusercontent.com/10243139/137590374-f85d79c7-d6a1-4cc5-8fbb-103425f49213.png)

- Create a new pipeline

- Select GitHub

<img width="1235" alt="Jenkins-Select-Github" src="https://user-images.githubusercontent.com/10243139/137590465-243ba7a0-d2e1-4218-92fb-e0c4aefcade7.png">

- Connect Jenkins with GitHub

<img width="1261" alt="Jenkins-Create-Access-Token-To-Github" src="https://user-images.githubusercontent.com/10243139/137590482-6bb60d31-d792-4873-ab8c-cf3b5a19823b.png">

- Login to GitHub & Generate an Access Token

![Jenkins-Github-Access-Token](https://user-images.githubusercontent.com/10243139/137590510-14f762c9-4a58-4551-a8cb-441a7cc05fe1.png)
<img width="1019" alt="Jenkins-Github-Generate-Token" src="https://user-images.githubusercontent.com/10243139/137590551-e43b78ed-e54c-4e1e-a794-f1368a143677.png">

- Copy Access Token

<img width="1185" alt="Jenkins-Copy-Token" src="https://user-images.githubusercontent.com/10243139/137590581-5ea60eed-04f7-4358-963c-af1fe30af15f.png">

- Paste the token and connect

<img width="1260" alt="JEnkins-Paste-Token-And-Connect" src="https://user-images.githubusercontent.com/10243139/137590598-e30f3f6b-9ab0-4eea-82ee-5e56d3335458.png">

- Create a new Pipeline

![1 o](https://user-images.githubusercontent.com/10243139/137590387-67732d4f-ea2e-496c-8c01-bf7c8f6eac82.png)

- Here is our newly created pipeline. It takes the name of your GitHub repository.

![1 p](https://user-images.githubusercontent.com/10243139/137590405-d32ae185-4912-49d2-8086-4a34031277a1.png)

Let us create our Jenkinsfile

Inside the Ansible project, create a new directory deploy and start a new file Jenkinsfile inside the directory.

Add the code snippet below to start building the Jenkinsfile gradually. This pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage

    pipeline {
        agent any

      stages {
        stage('Build') {
          steps {
            script {
              sh 'echo "Building Stage"'
            }
          }
        }
        }
    }

![1 r](https://user-images.githubusercontent.com/10243139/137590809-1c242ba4-d5b9-4d79-9d0e-304de99fc580.png)

Now go back into the Ansible pipeline in Jenkins, and select configure

![Jenkins-Select-Configure](https://user-images.githubusercontent.com/10243139/137590894-c221e266-82d2-4ff1-b5fa-10bf5a675040.png)

Scroll down to Build Configuration section and specify the location of the Jenkinsfile at deploy/Jenkinsfile

<img width="1434" alt="Jenkinsfile-Location" src="https://user-images.githubusercontent.com/10243139/137590935-93c44d00-088f-4d9e-978b-dbea5f087e6e.png">

Back to the pipeline again, this time click "Build now"

<img width="810" alt="Jenkins-Build-Now" src="https://user-images.githubusercontent.com/10243139/137591069-02b380a0-26f3-4715-a624-8d3e0e7630c4.png">

This will trigger a build and you will be able to see the effect of our basic Jenkinsfile configuration by going through the console output of the build.

![1 v](https://user-images.githubusercontent.com/10243139/137591097-3fd510f5-d254-4219-a27e-4710b9df69a9.png)

To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.

- Click on Blue Ocean
- Select your project
- Click on the play button against the branch

![1 w](https://user-images.githubusercontent.com/10243139/137591155-0d462f72-17bd-454d-8a12-ac70e79c931a.png)

### Add-On Bonus: Executing Multi-Branch Pipeline 

Notice that this pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch.

Let us see this in action.

Create a new git branch and name it feature/jenkinspipeline-stages

    Git checkout -b feature/jenkinspipeline-stages

Currently we only have the Build stage. Let us add another stage called Test. Paste the code snippet below and push the new changes to GitHub.

       pipeline {
        agent any

      stages {
        stage('Build') {
          steps {
            script {
              sh 'echo "Building Stage"'
            }
          }
        }

        stage('Test') {
          steps {
            script {
              sh 'echo "Testing Stage"'
            }
          }
        }
        }
    }

![1 x b](https://user-images.githubusercontent.com/10243139/137591317-5859dcd0-095a-4aec-a7ea-e7e0aa248a69.png)

To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.

- Click on the "Administration" button
- Navigate to the Ansible project and click on "Scan repository now"

![Jenkins-Scan-Repository-Now](https://user-images.githubusercontent.com/10243139/137591377-ab4434dc-b5c1-4066-afc2-c4f9cfe9706f.png)

Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.

![1 x d](https://user-images.githubusercontent.com/10243139/137591394-620ff58d-9106-49bf-8a77-063782f24f59.png)

In Blue Ocean, you can now see how the Jenkinsfile has caused a new step in the pipeline launch build for the new branch.

![1 x e](https://user-images.githubusercontent.com/10243139/137591408-a6cca3b7-f46f-4bac-89ca-8e1d71601be8.png)


## A QUICK TASK FOR YOU!

- Create a pull request to merge the latest code into the main branch
- After merging the PR, go back into your terminal and switch into the main branch.
- Pull the latest change.
- Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an echo command like we have in build and test stages)
    - Package 
    - Deploy 
    - Clean up
- Verify in Blue Ocean that all the stages are working, then merge your feature branch to the main branch
- Eventually, your main branch should have a successful pipeline like this in blue ocean

## SOLUTION

Create a pull request to merge the latest code into the main branch

    git checkout main
    git merge feature/jenkinspipeline-stages
    
![1 y a](https://user-images.githubusercontent.com/10243139/137591523-1dc81cc6-6f6f-4651-bef9-1f19eb34a072.png)

Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an echo command like we have in build and test stages)
   
- Package 
- Deploy 
- Clean up

        Git checkout -b feature/jenkinspipeline-morestages

Add the code below into the Jenkinsfile

       pipeline {
        agent any

      stages {
        stage('Build') {
          steps {
            script {
              sh 'echo "Building Stage"'
            }
          }
        }

        stage('Test') {
          steps {
            script {
              sh 'echo "Testing Stage"'
            }
          }
        }


        stage('Package') {
          steps {
            script {
              sh 'echo "Packaging Stage"'
            }
          }
        }

        stage('Deploy') {
          steps {
            script {
              sh 'echo "Deploying Stage"'
            }
          }
        }

        stage('Clean up') {
          steps {
            script {
              sh 'echo "Cleaning up Stage"'
            }
          }
        }
        }

    }

![1 y b](https://user-images.githubusercontent.com/10243139/137591587-f5989669-c122-4b1f-965d-066d08fab069.png)

Verify in Blue Ocean that all the stages are working, then merge your feature branch to the main branch

![1 y c](https://user-images.githubusercontent.com/10243139/137591669-66bef0b2-85ea-4718-8179-d38105b19a66.png)

Eventually, your main branch should have a successful pipeline like this in blue ocean

![1 y d](https://user-images.githubusercontent.com/10243139/137591678-524745c8-3b93-4ad0-bfff-941251e22404.png)


## Part 2 – Running Ansible Playbook from Jenkins

Now that you have a broad overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by:

Installing Ansible on Jenkins

    sudo apt install Ansible -y

![2 a](https://user-images.githubusercontent.com/10243139/137591976-81bbc93d-6b23-4f40-b6ad-2a8b6b8cd711.png)

Installing Ansible plugin in Jenkins UI

Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)


## CI/CD PIPELINE FOR TODO APPLICATION
We already have tooling website as a part of deployment through Ansible. Here we will introduce another PHP application to add to the list of software products we are managing in our infrastructure. The good thing with this particular application is that it has unit tests, and it is an ideal application to show an end-to-end CI/CD pipeline for a particular application.

Our goal here is to deploy the application onto servers directly from Artifactory rather than from git. If you have not updated Ansible with an Artifactory role, simply use this guide to create an Ansible role for Artifactory (ignore the Nginx part). Configure Artifactory on Ubuntu 20.04

- Phase 1 – Prepare Jenkins
Fork the repository below into your GitHub account
https://github.com/darey-devops/php-todo.git
On you Jenkins server, install PHP, its dependencies and Composer tool (Feel free to do this manually at first, then update your Ansible accordingly later)
    sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}
Install Jenkins plugins
Plot plugin
Artifactory plugin
We will use plot plugin to display tests reports, and code coverage information.
The Artifactory plugin will be used to easily upload code artifacts into an Artifactory server.
In Jenkins UI configure Artifactory


### Configure the server ID, URL and Credentials, run Test Connection.



- Phase 2 – Integrate Artifactory repository with Jenkins
Create a dummy Jenkinsfile in the repository
Using Blue Ocean, create a multibranch Jenkins pipeline
On the database server, create database and user
Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';
Update the database connectivity requirements in the file .env.sample
Update Jenkinsfile with proper pipeline configuration

```
pipeline {
    agent any

  stages {

     stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

    stage('Checkout SCM') {
      steps {
            git branch: 'main', url: 'https://github.com/darey-devops/php-todo.git'
      }
    }

    stage('Prepare Dependencies') {
      steps {
             sh 'mv .env.sample .env'
             sh 'composer install'
             sh 'php artisan migrate'
             sh 'php artisan db:seed'
             sh 'php artisan key:generate'
      }
    }
  }
}
```
Notice the Prepare Dependencies section

The required file by PHP is .env so we are renaming .env.sample to .env
Composer is used by PHP to install all the dependent libraries used by the application
php artisan uses the .env file to setup the required database objects – (After successful run of this step, login to the database, run show tables and you will see the tables being created for you)
Update the Jenkinsfile to include Unit tests step

```
    stage('Execute Unit Tests') {
      steps {
             sh './vendor/bin/phpunit'
      } 
 ```
 
- Phase 3 – Code Quality Analysis
This is one of the areas where developers, architects and many stakeholders are mostly interested in as far as product development is concerned. As a DevOps engineer, you also have a role to play. Especially when it comes to setting up the tools.

For PHP the most commonly tool used for code quality analysis is phploc. Read the article here for more

The data produced by phploc can be ploted onto graphs in Jenkins.

Add the code analysis step in Jenkinsfile. The output of the data will be saved in build/logs/phploc.csv file.

```
stage('Code Analysis') {
  steps {
        sh 'phploc app/ --log-csv build/logs/phploc.csv'

  }
}
```
Plot the data using plot Jenkins plugin.
This plugin provides generic plotting (or graphing) capabilities in Jenkins. It will plot one or more single values variations across builds in one or more plots. Plots for a particular job (or project) are configured in the job configuration screen, where each field has additional help information. Each plot can have one or more lines (called data series). After each build completes the plots’ data series latest values are pulled from the CSV file generated by phploc.

```
    stage('Plot Code Coverage Report') {
      steps {

            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)                          ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'A - Lines of code', yaxis: 'Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Directories,Files,Namespaces', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'B - Structures Containers', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Average Class Length (LLOC),Average Method Length (LLOC),Average Function Length (LLOC)', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'C - Average Length', yaxis: 'Average Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Cyclomatic Complexity / Lines of Code,Cyclomatic Complexity / Number of Methods ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'D - Relative Cyclomatic Complexity', yaxis: 'Cyclomatic Complexity by Structure'      
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Classes,Abstract Classes,Concrete Classes', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'E - Types of Classes', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Methods,Non-Static Methods,Static Methods,Public Methods,Non-Public Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'F - Types of Methods', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Constants,Global Constants,Class Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'G - Types of Constants', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Test Classes,Test Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'I - Testing', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Logical Lines of Code (LLOC),Classes Length (LLOC),Functions Length (LLOC),LLOC outside functions or classes ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'AB - Code Structure by Logical Lines of Code', yaxis: 'Logical Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Functions,Named Functions,Anonymous Functions', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'H - Types of Functions', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Interfaces,Traits,Classes,Methods,Functions,Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'BB - Structure Objects', yaxis: 'Count'

      }
    }
 ```
You should now see a Plot menu item on the left menu. Click on it to see the charts. (The analytics may not mean much to you as it is meant to be read by developers. So, you need not worry much about it – this is just to give you an idea of the real-world implementation).



- Bundle the application code for into an artifact (archived package) upload to Artifactory

```
stage ('Package Artifact') {
    steps {
            sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
     }
    }
 ```
Publish the resulted artifact into Artifactory

```
stage ('Upload Artifact to Artifactory') {
          steps {
            script { 
                 def server = Artifactory.server 'artifactory-server'                 
                 def uploadSpec = """{
                    "files": [
                      {
                       "pattern": "php-todo.zip",
                       "target": "<name-of-artifact-repository>/php-todo",
                       "props": "type=zip;status=ready"

                       }
                    ]
                 }""" 

                 server.upload spec: uploadSpec
               }
            }
        }
        
 ```
- Deploy the application to the dev environment by launching Ansible pipeline
```
stage ('Deploy to Dev Environment') {
    steps {
    build job: 'ansible-project/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
    }
  }
 ```
The build job used in this step tells Jenkins to start another job. In this case it is the ansible-project job, and we are targeting the main branch. Hence, we have ansible-project/main. Since the Ansible project requires parameters to be passed in, we have included this by specifying the parameters section. The name of the parameter is env and its value is dev. Meaning, deploy to the Development environment.

But how are we certain that the code being deployed has the quality that meets corporate and customer requirements? Even though we have implemented Unit Tests and Code Coverage Analysis with phpunit and phploc, we still need to implement Quality Gate to ensure that ONLY code with the required code coverage, and other quality standards make it through to the environments.

To achieve this, we need to configure SonarQube – An open-source platform developed by SonarSource for continuous inspection of code quality to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities.


##Screenshots
<img width="814" alt="Screenshot 2022-05-26 at 13 55 46" src="https://user-images.githubusercontent.com/33035619/170591547-57891108-d8eb-409d-bcd1-4fea8d4fc374.png">
<img width="1800" alt="Screenshot 2022-05-26 at 22 03 46" src="https://user-images.githubusercontent.com/33035619/170591558-e286bdea-5c0e-446d-aa84-a3a34cc4495f.png">
<img width="1800" alt="Screenshot 2022-05-26 at 22 03 46" src="https://user-images.githubusercontent.com/33035619/170591551-c555656d-882c-43b0-8f79-825dcbf267b7.png">
<img width="1800" alt="Screenshot 2022-05-26 at 22 55 36" src="https://user-images.githubusercontent.com/33035619/170591565-2d40f5d2-f64c-485b-93ff
<img width="1800" alt="Screenshot 2022-05-26 at 22 55 36" src="https://user-images.githubusercontent.com/33035619/170591573-a0d22862-5c94-42a0-a47a-1188d234ed83.png">
-fa713e8782bd.png">
<img width="1800" alt="Screenshot 2022-05-26 at 22 55 36" src="https://user-images.githubusercontent.com/33035619/170591606-66cb7093-223f-4726-8154-71bc4abc9cde.png">
<img width="756" alt="Screenshot 2022-05-26 at 23 41 55" src="https://user-images.githubusercontent.com/33035619/170591617-82422525-849f-4855-a1ce-8f7e19d62bb7.png">





Publish the resulted artifact into Artifactory

![image](https://user-images.githubusercontent.com/87030990/165833738-e4ecee5c-7a23-42cd-9861-e7f6bf6a2fa3.png)

![image](https://user-images.githubusercontent.com/87030990/165833822-0c90fda2-ceec-434d-a738-37dae3c803fc.png)

Deploy the application to the dev environment by launching Ansible pipeline

![image](https://user-images.githubusercontent.com/87030990/165833947-33d3d6ba-6d87-4c07-a16f-792f124a5512.png)

### SonarQube Installation

SonarQube was installed on Ubuntu EC2 instance using Ansible role with PostgreSQL as Backend Database using Ansible role

Java was installed on the EC2 instance for Sonarqube

Install OpenJDK and Java Runtime Environment (JRE) 11
```bash
sudo apt-get install openjdk-11-jdk -y
sudo apt-get install openjdk-11-jre -y
````
 
Set default JDK – To set default JDK or switch to OpenJDK enter below command:
````bash
sudo update-alternatives --config java
````

JAVA Version was verified with
````bash
java -version
````

PostgreSQL software was downloaded
````bash
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
````

PostgreSQL Database Server installed
````bash
sudo apt-get -y install postgresql postgresql-contrib
 ````
 
PostgreSQL Database Server was started and enabled to start automatically at boot time
````bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
````

Password for default postgres user was changed
````bash
sudo passwd postgres
````

Switch to the postgres user
````bash
su - postgres
````

A new user was created by typing
```bash
createuser sonar
````

Switch to the PostgreSQL shell
````bash
psql
````
 
Password was set for the newly created user for SonarQube database

````bash
ALTER USER sonar WITH ENCRYPTED password 'sonar';
````

A new database for PostgreSQL database was created by running:
````bash
CREATE DATABASE sonarqube OWNER sonar;
````

All privileges was granted to sonar user on sonarqube Database.
````bash
grant all privileges on DATABASE sonarqube to sonar;
````

Exit from the psql shell:
````bash
\q
````

Switch back to the sudo user by running the exit command.
````bash
exit
````

Sonarqube was installed using Ansible Playbook

![image](https://user-images.githubusercontent.com/87030990/165834870-c3d0689e-813b-4c11-8b64-828729bdfce5.png)

#### Access SonarQube

SonarQube was accessed using browser with default administrator username and password – admin
http://server_IP:9000 OR http://localhost:9000

![image](https://user-images.githubusercontent.com/87030990/165835067-c16ecc48-492a-46eb-af35-bb00d6cf2d4d.png)

SonarScanner plugin was installed in Jenkins and configure as shown

![image](https://user-images.githubusercontent.com/87030990/165835168-eb677c3d-8a53-4314-ab42-5c838176a3eb.png)

#### Configure SonarQube and Jenkins for Quality Gate

In Jenkins, install SonarScanner plugin and navigate to configure system in Jenkins. 

Add SonarQube server, setup SonarQube scanner from Jenkins and create webhook

![image](https://user-images.githubusercontent.com/87030990/165842874-aa9193d9-32e5-4750-aff2-2b17f0cfc98a.png)
![image](https://user-images.githubusercontent.com/87030990/165843158-6ce57412-7ef6-4a0d-a87f-34c06c05e564.png)
 
On SonarQube UI

![image](https://user-images.githubusercontent.com/87030990/165835482-66eea5ac-25a6-4420-a95d-71a930cb10ce.png)

Jenkins pipeline was updated with below snippet to include SonarQube scanning and Quality Gate
Below is the snippet for a Quality Gate stage in Jenkinsfile

````javascript
   stage('SonarQube Quality Gate') {
        environment {
            scannerHome = tool 'SonarQubeScanner'
        }
        steps {
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }

        }
    }
````

NOTE: The above step will fail because we have not updated `sonar-scanner.properties
 
Configure sonar-scanner.properties

 – From the step above, Jenkins will install the scanner tool on the Linux server. You will need to go into the tools directory on the server to configure the properties file in which SonarQube will require to function during pipeline execution.
 
 ````bash
 cd /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/conf/
 ````
 
 ![image](https://user-images.githubusercontent.com/87030990/165835812-76d48dc9-54eb-4ed7-a560-2ab945fdcb57.png)


Open sonar-scanner.properties File

````bash
sudo vi sonar-scanner.properties
````
````bash
sonar.host.url=http://<SonarQube-Server-IP-address>:9000
sonar.projectKey=php-todo
#----- Default source code encoding
sonar.sourceEncoding=UTF-8
sonar.php.exclusions=**/vendor/**
sonar.php.coverage.reportPaths=build/logs/clover.xml
sonar.php.tests.reportPath=build/logs/junit.xml
````

Add configuration related to php-todo Project

**Challenge:** The pipeline failed after updating sonar-scanner.properties file.
**Error message:** soar.source must be defined
![image](https://user-images.githubusercontent.com/87030990/165844043-db85d124-1e30-42a0-b8ca-824a4405d38b.png)

**Resolution:** sonar.sources was then added to the configuration.

````bash
sonar.host.url=http://<SonarQube-Server-IP-address>:9000
sonar.projectKey=php-todo
#----- Default source code encoding
sonar.sourceEncoding=UTF-8
sonar.php.exclusions=**/vendor/**
sonar.php.coverage.reportPaths=build/logs/clover.xml
sonar.php.tests.reportPath=build/logs/junit.xml
sonar.sources=/var/lib/jenkins/workspace/php-todo_main/
````

**HINT:** To know what exactly to put inside the 
sonar-scanner.properties file, SonarQube has a configurations page where you can get some directions.

![image](https://user-images.githubusercontent.com/87030990/165836419-507ea767-0369-4f48-8092-ff6d9a590ed3.png)

The quality gate we just included has no effect. Why? Well, because if you go to the SonarQube UI, you will realise that we just pushed a poor-quality code onto the development environment.Navigate to php-todo project in SonarQube
 
List the content to see the scanner tool sonar-scanner. That is what we are calling in the pipeline script.
Output of 
````bash
ls -latr
````

````bash
ubuntu@ip-172-31-16-176:/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/bin$ ls -latr
total 24
-rwxr-xr-x 1 jenkins jenkins 2550 Oct  2 12:42 sonar-scanner.bat
-rwxr-xr-x 1 jenkins jenkins  586 Oct  2 12:42 sonar-scanner-debug.bat
-rwxr-xr-x 1 jenkins jenkins  662 Oct  2 12:42 sonar-scanner-debug
-rwxr-xr-x 1 jenkins jenkins 1823 Oct  2 12:42 sonar-scanner
drwxr-xr-x 2 jenkins jenkins 4096 Dec 26 18:42 .
````

![image](https://user-images.githubusercontent.com/87030990/165836718-909d70c5-2882-4549-957a-5776e3431c21.png)

So far you have been given code snippets on each of the stages within the 
Jenkinsfile. But, you should also be able to generate Jenkins configuration code yourself.

* To generate Jenkins code, navigate to the dashboard for the php-todo pipeline and click on the Pipeline Syntax menu item
* Click on Steps and select withSonarQubeEnv
* – This appears in the list because of the previous SonarQube configurations you have done in Jenkins. Otherwise, it would not be there.

#### Conditionally deploy to higher environments

In the real world, developers will work on feature branches in a repository (e.g., GitHub or GitLab). There are other branches that will be used differently to control how software releases are done. You will see such branches as:
Develop
Master or Main 
(The * is a place holder for a version number, Jira Ticket name or some description. It can be something like Release-1.0.0)
Feature/*
Release/*
Hotfix/*
etc.

There is a very wide discussion around release strategy, and git branching strategies which in recent years are considered under what is known as GitFlow (Have a read and keep as a bookmark – it is a possible candidate for an interview discussion, so take it seriously!)

There is a very wide discussion around release strategy, and git branching strategies which in recent years are considered under what is known as GitFlow (Have a read and keep as a bookmark – it is a possible candidate for an interview discussion, so take it seriously!)
Assuming a basic gitflow implementation restricts only the develop branch to deploy code to Integration environment like sit
.
Let us update our Jenkinsfile to implement this:

First, we will include a When condition to run Quality Gate whenever the running branch is either develop, hotfix, release, main, or master
````javascript
when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP"}
````

Then we add a timeout step to wait for SonarQube to complete analysis and successfully finish the pipeline only when code quality is acceptable.
````javascript
timeout(time: 1, unit: 'MINUTES') {
        waitForQualityGate abortPipeline: true
    }
 
The complete stage will now look like this:
   stage('SonarQube Quality Gate') {
      when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP"}
        environment {
            scannerHome = tool 'SonarQubeScanner'
        }
        steps {
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
            }
            timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
    }
````

To test, create different branches and push to GitHub. You will realise that only branches other than develop, hotfix, release, main, or master will be able to deploy the code.

**Challenge:** A defect was encountered when sonar server URL was configured (in Configure System) on Jenkins  with a trailing slash (http://3.135.217.42:9000/) causing java.lang.IllegalStateException: “Unable to parse response from”

![image](https://user-images.githubusercontent.com/87030990/165837979-dfe6bf50-6a43-45d6-9082-2ed545ff24c1.png)

Resolution: The trailing slash in the sonarqube server (url http://3.135.217.42:9000) on Jenkins was removed to resolve the problem

![image](https://user-images.githubusercontent.com/87030990/165838154-2010ecf8-e508-4838-8822-22322e265fa2.png)

Notice that with the current state of the code, it cannot be deployed to Integration environments due to its quality. In the real world, DevOps engineers will push this back to developers to work on the code further, based on SonarQube quality report. Once everything is good with code quality, the pipeline will pass and proceed with sipping the codes further to a higher environment.

#### Complete the following tasks to finish Project 14
* Introduce Jenkins agents/slaves – Add 2 more servers to be used as Jenkins slave. Configure Jenkins to run its pipeline jobs randomly on any available slave nodes.

![image](https://user-images.githubusercontent.com/87030990/165838350-98879db4-13bb-4396-b319-ae4d3cfc721a.png)

* Configure webhook between Jenkins and GitHub to automatically run the pipeline when there is a code push.

![image](https://user-images.githubusercontent.com/87030990/165838445-0ac6f898-b4ec-4dd1-937f-6d486b2de624.png)

Deploy the application to all the environments

![image](https://user-images.githubusercontent.com/87030990/165848777-e40597e4-d8f5-44d6-8f5d-e703e7599c35.png)

![image](https://user-images.githubusercontent.com/87030990/165848726-4be3b358-695a-473f-ae0b-0b6d04af649b.png)


![image](https://user-images.githubusercontent.com/87030990/165848867-33703fc1-ddb3-44b0-9302-b871c428ac30.png)



