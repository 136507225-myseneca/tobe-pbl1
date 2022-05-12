# ANSIBLE CONFIGURATION MANAGEMENT – AUTOMATE PROJECT 7 TO 10

## Task
- Install and configure Ansible client to act as a Jump Server/Bastion Host
- Create a simple Ansible playbook to automate servers configuration

## INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE
Update Name tag on your Jenkins EC2 Instance to Jenkins-Ansible. We will use this server to run playbooks.
In your GitHub account create a new repository and name it ansible-config-mgt.
Instal Ansible
sudo apt update

```javascript
sudo apt install ansible
```
Check your Ansible version by running ansible --version




Configure Jenkins build job to save your repository content every time you change it – this will solidify your Jenkins configuration skills acquired in Project 9.
Create a new Freestyle project ansible in Jenkins and point it to your ‘ansible-config-mgt’ repository.
Configure Webhook in GitHub and set webhook to trigger ansible build.
Configure a Post-build job to save all (**) files, like you did it in Project 9.
Test your setup by making some change in README.MD file in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder

```javascript
ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/
```
Note: Trigger Jenkins project execution only for /main (master) branch.

Now your setup will look like this:



Tip Every time you stop/start your Jenkins-Ansible server – you have to reconfigure GitHub webhook to a new IP address, in order to avoid it, it makes sense to allocate an Elastic IP to your Jenkins-Ansible server (you have done it before to your LB server in Project 10). Note that Elastic IP is free only when it is being allocated to an EC2 Instance, so do not forget to release Elastic IP once you terminate your EC2 Instance.

-  # Step 2 – Prepare your development environment using Visual Studio Code
First part of ‘DevOps’ is ‘Dev’, which means you will require to write some codes and you shall have proper tools that will make your coding and debugging comfortable – you need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks, you can choose whichever you are comfortable with, but we recommend one free and universal editor that will fully satisfy your needs – Visual Studio Code (VSC), you can get it here.

After you have successfully installed VSC, configure it to connect to your newly created GitHub repository.

Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance

git clone <ansible-config-mgt repo link>
  
  


  
  
## INSTALL AND CONFIGURE JENKINS SERVER
- ### Step 1 – Install Jenkins server
Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

Install JDK (since Jenkins is a Java-based application)

 ```javascript 
sudo apt update
sudo apt install default-jdk-headless

  ```
Install Jenkins  
   ```javascript 
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
  ```
  
Make sure Jenkins is up and running

  ```javascript 
sudo systemctl status jenkins
  ```
  
By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group


Perform initial Jenkins setup.
From your browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080

You will be prompted to provide a default admin password



Retrieve it from your server:

   ```javascript 
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
  ```
Then you will be asked which plugings to install – choose suggested plugins.



Once plugins installation is done – create an admin user and you will get your Jenkins server address.

The installation is completed!

    ## ScreenShots 
  <img width="895" alt="Screenshot 2022-05-12 at 12 32 01" src="https://user-images.githubusercontent.com/33035619/168175732-348e3d39-038a-4d3f-ba37-5aa7edb7eaee.png">


- ### Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks
In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

Enable webhooks in your GitHub repository settings


Go to Jenkins web console, click "New Item" and create a "Freestyle project"


To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself



In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.



Save the configuration and let us try to run the build. For now we can only do it manually.
Click "Build Now" button, if you have configured everything correctly, the build will be successfull and you will see it under #1



You can open the build and check in "Console Output" if it has run successfully.

If so – congratulations! You have just made your very first Jenkins build!

But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

Click "Configure" your job/project and add these two configurations
Configure triggering the job from GitHub webhook:



Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts".



Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.



You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.

By default, the artifacts are stored on Jenkins server locally
  
 ```javascript 
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```
  
    ## ScreenShots 
<img width="888" alt="Screenshot 2022-05-12 at 17 15 15" src="https://user-images.githubusercontent.com/33035619/168175932-570f7346-4cf9-4991-84a5-67c53b4e9677.png">
<img width="1800" alt="Screenshot 2022-05-12 at 17 17 29" src="https://user-images.githubusercontent.com/33035619/168175942-0793c72c-28f6-4aa7-bdf2-2afb7d6726e4.png">
<img width="1800" alt="Screenshot 2022-05-12 at 17 17 40" src="https://user-images.githubusercontent.com/33035619/168175948-1a6aed0e-de87-4313-9cb3-17168c002adc.png">


  
  
# CREATE A COMMON PLAYBOOK
  
## Step 5 Create a Common Playbook
It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in inventory/dev.

In common.yml playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update your playbooks/common.yml file with following code:
  
```javascript 
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
  
 ```
Examine the code above and try to make sense out of it. This playbook is divided into two parts, each of them is intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on your RHEL 8 and Ubuntu servers. It uses root user to perform this task and respective package manager: yum for RHEL 8 and apt for Ubuntu.

Feel free to update this playbook with following tasks:

Create a directory and a file inside it
Change timezone on all servers
Run some shell script
…
For a better understanding of Ansible playbooks – watch this video from RedHat and read this article.

  ## ScreenShots 
  <img width="1800" alt="Screenshot 2022-05-12 at 17 03 28" src="https://user-images.githubusercontent.com/33035619/168176073-ec400722-b0cb-4f9b-8df5-157f232bea05.png">
<img width="888" alt="Screenshot 2022-05-12 at 17 07 25" src="https://user-images.githubusercontent.com/33035619/168176119-0745b958-e7c3-4202-a9a2-4e682fdb709f.png">


  
  
  
  
  
## Step 6  Update GIT with the latest code
Now all of your directories and files live on your machine and you need to push changes made locally to GitHub.

In the real world, you will be working within a team of other DevOps engineers and developers. It is important to learn how to collaborate with help of GIT. In many organisations there is a development rule that do not allow to deploy any code before it has been reviewed by an extra pair of eyes – it is also called "Four eyes principle".

Now you have a separate branch, you will need to know how to raise a Pull Request (PR), get your branch peer reviewed and merged to the master branch.

Commit your code into GitHub:

use git commands to add, commit and push your branch to GitHub
  
```javascript 
git status

git add <selected files>

git commit -m "commit message"
```
  
Create a Pull request (PR)
  
  
  
  ## Screenshots
  <img width="884" alt="Screenshot 2022-05-12 at 17 14 56" src="https://user-images.githubusercontent.com/33035619/168176228-a6d0d84e-76bf-43ed-9fbf-09052b4f7aa1.png">
<img width="888" alt="Screenshot 2022-05-12 at 17 15 15" src="https://user-images.githubusercontent.com/33035619/168176241-26fc5666-a67a-485e-86aa-79cca3e2fb3b.png">
<img width="1800" alt="Screenshot 2022-05-12 at 17 17 29" src="https://user-images.githubusercontent.com/33035619/168176248-627b59b2-3470-4c64-bcf5-ffda796ecb47.png">



Wear a hat of another developer for a second, and act as a reviewer.

If the reviewer is happy with your new feature development, merge the code to the master branch.

Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.

Once your code changes appear in master branch – Jenkins will do its job and save all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server.


## RUN FIRST ANSIBLE TEST
### Step 7 Run first Ansible test
Now, it is time to execute ansible-playbook command and verify if your playbook actually works:

 ```javascript 
cd ansible-config-mgt
ansible-playbook -i inventory/dev.yml playbooks/common.yml
 ```
You can go to each of the servers and check if wireshark has been installed by running which wireshark or wireshark --version



Your updated with Ansible architecture now looks like this:


  ## Screenshots
  
<img width="559" alt="Screenshot 2022-05-12 at 16 27 19" src="https://user-images.githubusercontent.com/33035619/168176516-64554555-213b-4000-8f67-3c5ea5456447.png">

<img width="559" alt="Screenshot 2022-05-12 at 16 27 19" src="https://user-images.githubusercontent.com/33035619/168176388-66fde20b-a0c4-4534-ab55-b94384a96998.png">
 Screenshot 2022-05-12 at 22.54.38.png…]()
<img width="1002" alt="Screenshot 2022-05-12 at 22 54 48" src="https://user-images.githubusercontent.com/33035619/168176340-593ea102-cac4-4035-85b5-1fdcaf69e0c8.png">
<img width="820" alt="Screenshot 2022-05-12 at 22 58 15" src="https://user-images.githubusercontent.com/33035619/168176343-15570958-0351-4494-b4fd-1e05324ce9d1.png">
<img width="996" alt="Screenshot 2022-05-12 at 20 41 12" src="https://user-images.githubusercontent.com/33035619/168176354-539f7e50-8b6b-4cc6-9aeb-c9d079c0fa6e.png">


  

Optional step – Repeat once again
Update your ansible playbook with some new Ansible tasks and go through the full checkout -> change codes -> commit -> PR -> merge -> build -> ansible-playbook cycle again to see how easily you can manage a servers fleet of any size with just one command!

Congratulations
You have just automated your routine tasks by implementing your first Ansible project! There is more exciting projects ahead, so lets keep it moving!



