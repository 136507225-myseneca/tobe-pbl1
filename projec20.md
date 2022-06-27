# MIGRATION TO THE CLOUD AND CONTAINERIZATION PART 1: DOCKER AND DOCKER COMPOSE

## Install Docker and Prepare for Migration to the Cloud
- Spin up an Ubuntu t2.medium instance
- Paste in the following block of code in the terminal
  ```sh
  curl -fsSL https://get.docker.com -o get-docker.sh
  sh ./get-docker.sh
  usermod -aG docker ubuntu
  ```
## MySQL in Container
- Pull MySQL Docker image from Docker Hub
  ```sh
  docker pull mysql/mysql-server:latest
  ```
 <img width="871" alt="Screenshot 2022-06-27 at 11 31 46" src="https://user-images.githubusercontent.com/33035619/175921882-2ca4ea87-1a56-4581-84a9-f6af34c4a2cf.png">

  To list the images you have on your instance, type `docker image ls`

- Deploy the MySQL Container
  ```sh
  docker run --name mysql -e MYSQL_ROOT_PASSWORD=adminPassword -d mysql/mysql-server:latest
  ```
  You can change the **name** and **password** flags to suit your preference.
  Run `docker ps -a` to see all your containers (running or stopped)

## Connecting to the MySQL Docker container
- Connecting directly
  ```sh
  docker exec -it  mysql -u root -p
  ```
<img width="976" alt="Screenshot 2022-06-27 at 11 32 11" src="https://user-images.githubusercontent.com/33035619/175922247-d7d8b1bb-dc91-4fff-96ab-930757d5e380.png">
- Second approach
  - Create a network
    ```sh
    docker network create --subnet=172.18.0.0/24 tooling_app_network
    ```
  - Export an environment variable containing the root password setup earlier
    ```sh
    export MYSQL_PW=adminPassword
    ```
  - Run the container using the network created
    ```sh
    docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:latest
    ```
- Create a MySQL user using a script
  - Create a **create_user.sql** file and paste in the following
    ```sql
    CREATE USER ''@'%' IDENTIFIED BY ''; GRANT ALL PRIVILEGES ON * . * TO ''@'%';
    ```
  - Run the script
    ```sh
    docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < ./create_user.sql
    ```

- Run the MySQL client container
  ```sh
  docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u  -p 
  ```

## Prepare Database Schema
- Clone the Tooling app repository
  ```
  git clone https://github.com/darey-devops/tooling.git
  ```
- Export the location of the SQL file
  ```
  export tooling_db_schema=./tooling/html/tooling_db_schema.sql
  ```
- Use the script to create the database and prepare the schema
  ```
  docker exec -i mysql-server mysql -u root -p $MYSQL_PW < $tooling_db_schema
  ```
- Update *db_conn.php* file with database information
  ```
   $servername = "mysqlserverhost"; $username = ""; $password = ""; $dbname = "toolingdb"; 
   ```
- Run the Tooling app
  - First build the Docker image. Cd into the tooling folder, where the Dockerfile is and run
    ```
    docker build -t tooling:0.0.1 .
    ```
  - Run the container
    ```
    docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1
    ```
  <img width="943" alt="Screenshot 2022-06-27 at 11 32 26" src="https://user-images.githubusercontent.com/33035619/175922394-8f59eb1a-38cc-4595-8184-7d8639e355e8.png">
<img width="945" alt="Screenshot 2022-06-27 at 11 32 38" src="https://user-images.githubusercontent.com/33035619/175922500-08f47cc2-8b56-4ff1-9672-b51745079c02.png">

## Practice Task 1
### Implement a POC to migrate the PHP-Todo app into a containerized application.
### Part 1
- Clone the php-todo repo here: https://github.com/Anefu/php-todo.git 
- Write a Dockerfile for the TODO application
  ```
  FROM php:7.4-alpine

  RUN apk add git &>/dev/null

  RUN git clone https://github.com/darey-devops/php-todo.git $HOME/php-todo &>/dev/null

  COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

  RUN install-php-extensions pdo_mysql &>/dev/null && install-php-extensions @composer &>/dev/null

  ENV DB_HOST=db

  WORKDIR /root/php-todo

  RUN sed -i -e "s/DB_HOST=127\.0\.0\.1/DB_HOST=${DB_HOST}/" .env.sample && mv .env.sample .env 

  RUN composer install --ignore-platform-reqs &>/dev/null

  COPY ./serve.sh serve.sh

  ENTRYPOINT ["sh", "serve.sh"]
  ```
  the serve.sh file should contain the following block:
  ```
  php artisan migrate
  php artisan key:generate
  php artisan db:seed
  php artisan serve --host:0.0.0.0
  ```
- Run both database and app on your Docker Engine
<img width="963" alt="Screenshot 2022-06-27 at 11 33 02" src="https://user-images.githubusercontent.com/33035619/175922605-1d7ecf2e-3a33-45da-8b1d-6dbe566bc3b7.png">
- Access the application from the browser
<img width="979" alt="Screenshot 2022-06-27 at 11 33 11" src="https://user-images.githubusercontent.com/33035619/175922628-fc3bda05-843e-4eea-a8c8-dc6fbb129106.png">

### Part 2
- Create an account on Docker Hub
)
- Create a Docker Hub repository

- Push the docker images from the instance to Docker Hub
  - First run `docker login` and enter your docker credentials.
  - Then retag your image and push:
    ```
    docker tag <tag> <repo-name>/<tag>
    docker push <repo-name>/<tag>
    ```
<img width="976" alt="Screenshot 2022-06-27 at 11 42 44" src="https://user-images.githubusercontent.com/33035619/175924989-e8cbac94-0176-4bac-a240-04aa62782b32.png">



### Part 3
- Write a Jenkinsfile for Docker build and push to registry
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
                git branch: 'main', url: 'https://github.com/Anefu/php-todo.git'
            }
        }
        
        stage('Build image') {
            steps {
                sh "docker build -t anefu/php-todo:${env.BRANCH_NAME}-${env.BUILD_NUMBER} ."
            }
        }
         stage('Docker Push') {
             when { expression { response.status == 200 } }
             steps {
                 withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                     sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                     sh "docker push anefu/php-todo:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                 }
             }
          }
      }
  }
  ```
- Connect the repo to Jenkins (using Blue Ocean plugin)
- Create a multibranch pipeline
- Simulate a Docker push
<img width="976" alt="Screenshot 2022-06-27 at 11 43 02" src="https://user-images.githubusercontent.com/33035619/175925315-ac44bf1b-411f-4f31-9a92-a0aaafa4e492.png">
- Verify the images can be found in the registry
<img width="958" alt="Screenshot 2022-06-27 at 11 51 11" src="https://user-images.githubusercontent.com/33035619/175925366-9ce5bd8a-991e-4f2f-bbd2-d15b4e42ec3c.png">

### Deployment with Docker Compose
- Install Docker Compose (https://docs.docker.com/compose/install/)
- Create a file, `tooling.yaml` and paste in the following block
  ```
  version: "3.9"
  services:
  tooling_frontend:
    build: .
    ports:
      - "5000:80"
    volumes:
      - tooling_frontend:/var/www/html
    links:
      - db
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: <The database name required by Tooling app >
      MYSQL_USER: <The user required by Tooling app >
      MYSQL_PASSWORD: <The password required by Tooling app >
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql
  volumes:
    tooling_frontend:
    db:
    ```
- Run the command `docker-compose -f tooling.yaml up -d` to start the containers.
- Verify the containers are running `docker-compose -f tooling.yaml ps`

## Practice Task 2
### 1. Document understanding of the various fields in `tooling.yaml`
- version: "3.9" specifies the version of the docker-compose API 
- services: defines configurations that are applied to containers when `docker-compose up` is run. 
  - tooling_frontend specifies the name of the first service.
  - build tells docker-compose that it needs to build and image from a Dockerfile for this service.
  - ports attaches port 5000 on the instance to port 80 on the container
  - volumes attaches a path on the host instance to any container created for the service
  - links connects one container to another (tooling_frontend to db in this case)
  - db defines the database service (can be any name)
  - image specifies the image to use for the containers, if it isn't available on the instance, it is pulled from Docker Hub
  - restart tells the container how frequently to restart
  - environment is used to pass environment variables required for the service running in the container


### 2. Update Jenkinsfile with a test stage
- Add the following after the **Build Image** stage in your Jenkinsfile for php-todo
  ```
        stage("Start the app") {
            steps {
                sh "docker-compose up -d"
            }
        }
        stage("Test endpoint") {
            steps {
                script {
                    while (true) {
                        def response = httpRequest 'http://localhost:8000'
                        if (response.status == 200) {
                            withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                                sh "docker push anefu/php-todo:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                            }
                            break 
                        }
                    }
                }
            }
        }
        stage('Docker Push') {
             when { expression { response.status == 200 } }
             steps {
                 withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                     sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                     sh "docker push anefu/php-todo:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                 }
             }
        }
  ```
  The "Test endpoint" stage uses httpRequest to check if the app can be reached after running `docker-compose up -d`.
<img width="976" alt="Screenshot 2022-06-27 at 11 51 50" src="https://user-images.githubusercontent.com/33035619/175925380-55aa5e4c-aac4-4d7c-b48b-4e20c8460b1a.png">
- Clean Up stage
  ```
  stage ("Remove images") {
    steps {
        sh "docker-compose down"
        sh "docker system prune -af"
    }
  }
  ```
<img width="978" alt="Screenshot 2022-06-27 at 11 51 59" src="https://user-images.githubusercontent.com/33035619/175925398-5a99f5a8-75d7-4a52-be0a-7749b4dad36c.png">

Link to php-todo repo: https://github.com/tobesegun/todo-app-with-docker-and-jenkins
