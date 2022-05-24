# ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES
```javascript
import = Static
include = Dynamic
```
- When the import module is used, all statements are pre-processed at the time playbooks are parsed. Meaning, when you execute site.yml playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

- On the other hand, when include module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used.

# INTRODUCING DYNAMIC ASSIGNMENT INTO OUR STRUCTURE

### Introducing Dynamic Assignment Into Our structure
In your https://github.com/<your-name>/ansible-config-mgt GitHub repository start a new branch and call it dynamic-assignments.

- Create a new folder, name it dynamic-assignments. Then inside this folder, create a new file and name it env-vars.yml. We will instruct site.yml to include this playbook later. For now, let us keep building up the structure.

- Your GitHub shall have following structure by now.

- Note: Depending on what method you used in the previous project you may have or not have roles folder in your GitHub repository – if you used ansible-galaxy, then roles directory was only created on your Jenkins-Ansible server locally. It is recommended to have all the codes managed and tracked in GitHub, so you might want to recreate this structure manually in this case – it is up to you.

```javascript
  
├── dynamic-assignments
│   └── env-vars.yml
├── inventory
│   └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
└── roles (optional folder)
    └──...(optional subfolders & files)
└── static-assignments
    └── common.yml
  
 ```
  
  
 # Screenshorts
  <img width="256" alt="Screenshot 2022-05-23 at 17 09 01" src="https://user-images.githubusercontent.com/33035619/169875151-1806678a-7746-4510-bb50-3dedda966a3a.png">

  
Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.

For this reason, we will now create a folder to keep each environment’s variables file. Therefore, create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables.

Your layout should now look like this.
  
```javascript

├── dynamic-assignments
│   └── env-vars.yml
├── env-vars
    └── dev.yml
    └── stage.yml
    └── uat.yml
    └── prod.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
├── playbooks
    └── site.yml
└── static-assignments
    └── common.yml
    └── webservers.yml
  
```
Now paste the instruction below into the env-vars.yml file.

```javascript
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
 ```
Notice 3 things to notice here:

We used include_vars syntax instead of include, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are:
- include_role
- include_tasks
- include_vars
In the same version, variants of import were also introduces, such as:

- import_role
- import_tasks
  
- We made use of a special variables { playbook_dir } and { inventory_file }. { playbook_dir } will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. { inventory_file } on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.
- We are including the variables using a loop. with_first_found implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

  ## Screenshots
  <img width="888" alt="Screenshot 2022-05-23 at 18 22 17" src="https://user-images.githubusercontent.com/33035619/169875326-b829c201-4ee9-4e46-a787-516173e30b2b.png">
  <img width="890" alt="Screenshot 2022-05-23 at 17 11 32" src="https://user-images.githubusercontent.com/33035619/169876393-4e715586-f68c-485f-bdf0-a9f7976d67a4.png">
  <img width="293" alt="Screenshot 2022-05-23 at 18 29 15" src="https://user-images.githubusercontent.com/33035619/169876526-0c6091e7-3fd3-4f3a-9d50-2c135b406d86.png">

  
## UPDATE SITE.YML WITH DYNAMIC ASSIGNMENTS

Update site.yml file to make use of the dynamic assignment. (At this point, we cannot test it yet. We are just setting the stage for what is yet to come. So hang on to your hats)

site.yml should now look like this.

```javascript
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
```
  
### Community Roles
Now it is time to create a role for MySQL database – it should install the MySQL package, create a database and configure users. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.

### Download Mysql Ansible Role
You can browse available community roles here

We will be using a MySQL role developed by geerlingguy.

Hint: To preserve your your GitHub in actual state after you install a new role – make a commit and push to master your ‘ansible-config-mgt’ directory. Of course you must have git installed and configured on Jenkins-Ansible server and, for more convenient work with codes, you can configure Visual Studio Code to work with this directory. In this case, you will no longer need webhook and Jenkins jobs to update your codes on Jenkins-Ansible server, so you can disable it – we will be using Jenkins later for a better purpose.

On Jenkins-Ansible server make sure that git is installed with git --version, then go to ‘ansible-config-mgt’ directory and run

```javascript
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
 ```
  
Inside roles directory create your new MySQL role with ansible-galaxy install geerlingguy.mysql and rename the folder to mysql

mv geerlingguy.mysql/ mysql
Read README.md file, and edit roles configuration to use correct credentials for MySQL required for the tooling website.

Now it is time to upload the changes into your GitHub:

```javascript
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
 ```
  
## Screenshot
  <img width="302" alt="Screenshot 2022-05-24 at 20 13 54" src="https://user-images.githubusercontent.com/33035619/170114575-c8a7613a-6812-45fb-98c2-5fa87caf8130.png">


Now, if you are satisfied with your codes, you can create a Pull Request and merge it to main branch on GitHub.

## LOAD BALANCER ROLES
We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:

Nginx
Apache
With your experience on Ansible so far you can:

Decide if you want to develop your own roles, or find available ones from the community
Update both static-assignment and site.yml files to refer the roles
Important Hints:

Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one – this is where you can make use of variables.

Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively.

Set both values to false like this enable_nginx_lb: false and enable_apache_lb: false.

Declare another variable in both roles load_balancer_is_required and set its value to false as well

Update both assignment and site.yml files respectively

loadbalancers.yml file

 ```javascript
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
site.yml file

     - name: Loadbalancers assignment
       hosts: lb
         - import_playbook: ../static-assignments/loadbalancers.yml
        when: load_balancer_is_required 
 ```
  
Now you can make use of env-vars\uat.yml file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.

You will activate load balancer, and enable nginx by setting these in the respective environment’s env-vars file.

enable_nginx_lb: true
load_balancer_is_required: true
The same must work with apache LB, so you can switch it by setting respective environmental variable to true and other to false.

To test this, you can update inventory for each environment and run Ansible against each environment.

  
##ScreenShots
  <img width="1800" alt="Screenshot 2022-05-24 at 13 17 23" src="https://user-images.githubusercontent.com/33035619/170115070-33fb6308-ca7b-4f5b-ae85-b2ba482e0419.png">
<img width="910" alt="Screenshot 2022-05-24 at 15 15 34" src="https://user-images.githubusercontent.com/33035619/170115098-e78010b1-5f98-4eb0-a58c-400646e22bb3.png">
<img width="1800" alt="Screenshot 2022-05-24 at 20 19 21" src="https://user-images.githubusercontent.com/33035619/170115385-bd332089-d8d1-4a82-9405-28270b1e4daf.png">

<img width="1800" alt="Screenshot 2022-05-24 at 20 19 26" src="https://user-images.githubusercontent.com/33035619/170115401-33008438-b3e8-4642-8776-ab2dc366e3dd.png">
enshot 2022-05-24 at 20.19.26.png…]()
<img width="1800" alt="Screenshot 2022-05-24 at 20 19 33" src="https://user-images.githubusercontent.com/33035619/170115438-edeba807-129e-4edd-8937-ac39ffdd8214.png">
<img width="1800" alt="Screenshot 2022-05-24 at 20 19 01" src="https://user-images.githubusercontent.com/33035619/170116072-a4ec4ea9-8349-4094-94d9-8657638005e4.png">
<img width="1800" alt="Screenshot 2022-05-24 at 20 18 46" src="https://user-images.githubusercontent.com/33035619/170116102-80896b16-005c-4cf5-91b1-6ab85b4e3fc3.png">
<img width="1800" alt="Screenshot 2022-05-24 at 20 39 41" src="https://user-images.githubusercontent.com/33035619/170124005-19e9b552-531b-4311-8628-657c366ddb51.png">
370-27c4-412a-8dea-f52c804b8e82.png">
<img width="1656" alt="Screenshot 2022-05-24 at 21 13 57" src="https://user-images.githubusercontent.com/33035619/170124013-e14bd0e7-82ac-4653-afb6-ff04ace01517.png">
<img width="1800" alt="Screenshot 2022-05-24 at 20 39 41" src="https://user-images.githubusercontent.com/33035619/170124577-db94f963-41f5-44d8-8877-047231dc5777.png">


 
Congratulations!
You have learned and practiced how to use Ansible configuration management tool to prepare UAT environment for Tooling web solution.

