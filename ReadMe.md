## TP1 - Hugo MONTAGNON

### 1-1. Document your database

#### Installation

1. **Créer un Réseau Docker** :
   ```sh
   docker network create app-network

2. **Construire l'Image Docker :** :
   ```sh
    docker build -t my-postgres-image .

3. **Démarrer le Conteneur PostgreSQL** :
   ```sh
   docker run -d \
    --name my-postgres-container \ #container name
    --network app-network \ #network
    -e POSTGRES_DB=db \ #database
    -e POSTGRES_USER=usr \ #user 
    -e POSTGRES_PASSWORD=pwd \ #password
    -v postgres_data:/var/lib/postgresql/data \ #volume
    my-postgres-image #ref image

4. **Démarrer Adminer** :
    ```sh
    docker run -d \ 
    --name adminer \ #container name
    --network app-network \ #network
    -p 8090:8080 \ #port
    adminer #image


### Fichiers necessaires pour créer l'image:

1. **Dockerfile** :

    ```sh
    #Utiliser l'image de base postgres:14.1-alpine
    FROM postgres:14.1-alpine

    #Définir les variables d'environnement pour PostgreSQL
    ENV POSTGRES_DB=db \
        POSTGRES_USER=usr \
        POSTGRES_PASSWORD=pwd

    #Copier les scripts SQL dans le répertoire d'initialisation de PostgreSQL
    COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/
    COPY 02-InsertData.sql /docker-entrypoint-initdb.d/

2. **CreateScheme.sql** :
    ```sql
        CREATE TABLE public.departments
    (
    id      SERIAL      PRIMARY KEY,
    name    VARCHAR(20) NOT NULL
    );

    CREATE TABLE public.students
    (
    id              SERIAL      PRIMARY KEY,
    department_id   INT         NOT NULL REFERENCES departments (id),
    first_name      VARCHAR(20) NOT NULL,
    last_name       VARCHAR(20) NOT NULL
    );

3. **CreateScheme.sql** :
    ```sql
    INSERT INTO departments (name) VALUES ('IRC');
    INSERT INTO departments (name) VALUES ('ETI');
    INSERT INTO departments (name) VALUES ('CGP');

    INSERT INTO students (department_id, first_name, last_name) VALUES (1, 'Eli', 'Copter');
    INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Emma', 'Carena');
    INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Jack', 'Uzzi');
    INSERT INTO students (department_id, first_name, last_name) VALUES (3, 'Aude', 'Javel');


### 1-2. Why do we need a multistage build? And explain each step of this dockerfile.

A multistage build in Docker allows you to use multiple `FROM` statements in a single Dockerfile, each representing a separate build stage. This is useful for optimizing Docker images by separating the build environment from the runtime environment. It helps reduce the size of the final image by discarding unnecessary build dependencies and artifacts.

 ```sh
    # Build
    FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
    ENV MYAPP_HOME /opt/myapp
    WORKDIR $MYAPP_HOME
    COPY pom.xml .
    COPY src ./src
    RUN mvn package -DskipTests

    # Run
    FROM amazoncorretto:17
    ENV MYAPP_HOME /opt/myapp
    WORKDIR $MYAPP_HOME
    COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

    ENTRYPOINT java -jar myapp.jar
```

#### Explanation of each step in the Dockerfile:

1. **Build Stage (myapp-build):**
   - `FROM maven:3.8.6-amazoncorretto-17 AS myapp-build`: Specifies the base image for the build stage, which includes Maven and JDK 17.
   - `ENV MYAPP_HOME /opt/myapp`: Sets an environment variable `MYAPP_HOME` to the directory `/opt/myapp`.
   - `WORKDIR $MYAPP_HOME`: Sets the working directory inside the container to `/opt/myapp`.
   - `COPY pom.xml .`: Copies the `pom.xml` file from the local directory to the working directory in the container.
   - `COPY src ./src`: Copies the source code files from the local `src` directory to the `src` directory in the container.
   - `RUN mvn package -DskipTests`: Builds the Maven project, skipping tests, to generate the application JAR file.

2. **Deployment Stage:**
   - `FROM amazoncorretto:17`: Specifies the base image for the deployment stage, which includes JDK 17.
   - `ENV MYAPP_HOME /opt/myapp`: Sets an environment variable `MYAPP_HOME` to the directory `/opt/myapp`.
   - `WORKDIR $MYAPP_HOME`: Sets the working directory inside the container to `/opt/myapp`.
   - `COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar`: Copies the application JAR file generated in the build stage to the working directory in the deployment stage.
   - `ENTRYPOINT java -jar myapp.jar`: Specifies the default command to run when the container starts, which is to execute the application JAR file using Java.

This Dockerfile uses a multistage build to separate the build process (compilation and packaging of the application) from the deployment process (running the application). The build stage uses a Maven-based image to compile the source code and generate the application JAR file, while the deployment stage uses a lightweight JDK image to run the application. This helps create a smaller and more efficient Docker image by discarding unnecessary build dependencies and intermediate files.


### 1-3 & 1-4 Configuration of the docker-compose.yml file

The [docker-compose.yml](docker-compose.yml) file defines the configuration for all the application's services. Here is an explanation of each section:

**Services**

- `backend`: This service builds and runs the Java backend application from the Dockerfile in [backend/simple-api-student-main/Dockerfile](backend/simple-api-student-main/Dockerfile). It depends on the `database` service and exposes port 8080.

- `database`: This service builds and runs a PostgreSQL database from the Dockerfile in [DB/Dockerfile](DB/Dockerfile). It exposes the `postgresdata` volume for data persistence.

- `httpd`: This service builds and runs an HTTP server from the Dockerfile in [frontend/Dockerfile](frontend/Dockerfile). It depends on the `backend` service and exposes port 80.

**Networks**

- `app-network`: A Docker network that allows the services to communicate with each other.

**Volumes**

- `postgresdata`: A Docker volume for persisting the PostgreSQL database data.

### How to use docker-compose

To start all the services, run the following command at the root of the project:

```sh
docker-compose up --build
```

### 1-5 Document your publication commands and published images in dockerhub.

1. **Publication des Images Docker sur Docker Hub**

Vos images Docker sont stockées localement, publions-les pour qu'elles puissent être utilisées par d'autres membres de l'équipe ou sur d'autres machines.

Vous aurez besoin d'un compte Docker Hub.

#### Étape 1 : Login

To connect to your Docker Hub account, use the following command:

```sh
docker login
```

#### Étape 2 : Tag Your Image

So far, we've only used the latest tag. Now that we want to publish our image, let's add a meaningful version to our images. Replace USERNAME with your Docker Hub username.

```sh
docker tag my-database USERNAME/my-database:1.0
```

#### Étape 3 : Push your image

To push an image to Docker Hub, use the following command:

```sh
docker push USERNAME/my-database:1.0
```

## TP2 - Hugo MONTAGNON

### Goals

This project aims to set up a complete pipeline workflow for testing and delivering a software application using various tools to build, test automatically, and check code quality.

### Setup GitHub Actions

Most of the CI services use a yaml file (except Jenkins that uses a… Groovy file…) to describe the expected steps to be done over the pipeline execution. Go on and create your first main.yml file into your project’s root directory.

For those who are not familiar with Maven and Java project structures, here is the command for building and running your tests:

```sh
mvn clean verify --files /path/to/pom.xml
```

***PS: Installation on Windows is a little more complicated to run locally. You'll need to download maven from their site and add the /bin path to the PC environment variables.***

This allows us to build and run local unit and integration tests directly inside your cash. 

**- First you create a .github/workflows directory in your repository on GitHub**

Put your main.yml inside workflows.

The main.yml holds the architecture of your pipeline. Each job will represent a step of what you want to do. Each job will be run in parallel unless a link is specified.

```yml
name: CI devops 2024

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  test-backend:
    runs-on: ubuntu-22.04
    env:
      DOCKER_USERNAME: ${{ secrets.USR_DOCKERHUB }}
      DOCKER_PASSWORD: ${{ secrets.PWD_DOCKERHUB }}
    steps:
      - name: Checkout du code GitHub
        uses: actions/checkout@v2.5.0

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Construction et test avec Maven
        #run: mvn clean verify --file backend/simple-api-student-main
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=devops-tp2-2024_tp2 -Dsonar.organization=devops-tp2-2024 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file backend/simple-api-student-main

  
```
Now, as soon as you push the project into github, github action will launch and run unit and integration tests. If everything is validated and turns green, it's OK. If not, you'll need to check for errors in the github actions tab.

### Setup pipline to save images in dockerhub

We're now going to complete our `main.yml` so that we can push our images, which we've just put on github, to dockerhub automatically. To do this, we'll add this section to our `main.yml`:

```yml
build-and-push-docker-images:
    runs-on: ubuntu-22.04
    needs: test-backend

    steps:
      - name: Récupérer le code
        uses: actions/checkout@v2

      - name: Login to DockerHub
        run: docker login -u ${{ secrets.USR_DOCKERHUB }} -p ${{ secrets.PWD_DOCKERHUB }}

      - name: Construire et pousser l'image backend
        uses: docker/build-push-action@v3
        with:
          context: ./backend/simple-api-student-main # Chemin vers le répertoire contenant le Dockerfile backend
          tags: ${{ secrets.USR_DOCKERHUB }}/tp1-backend:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Construire et pousser l'image de la base de données
        uses: docker/build-push-action@v3
        with:
          context: ./DB # Chemin vers le répertoire contenant le Dockerfile de la base de données
          tags: ${{ secrets.USR_DOCKERHUB }}/tp1-database:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Construire et pousser l'image httpd
        uses: docker/build-push-action@v3
        with:
          context: ./frontend # Chemin vers le répertoire contenant le Dockerfile httpd
          tags: ${{ secrets.USR_DOCKERHUB }}/tp1-httpd:latest
          push: ${{ github.ref == 'refs/heads/main' }}
```
***/!\ Don't forget to define the secret variables in github: `USR_DOCKERHUB` and `PWD_DOCKERHUB` so that the connection to the dockerhub account can be made.***

### Setup Quality Gate

Quality is here to make sure your code will be maintainable and determine every unsecured block. It helps you produce better and tested features, and it will also prevent having dirty code pushed inside your main branch.

For this purpose, we are going to use SonarCloud, a cloud solution that makes analysis and reports of your code. This is a useful tool that everyone should use in order to learn java best practices.

To do this, you must follow these different steps: 

- You must create an organization.

- And keep the project key and the organization key you will need it later.

- You need to add this script to your main.yml for launch sonar at each commit with this line:

```yml
run: mvn -B verify sonar:sonar -Dsonar.projectKey=devops-tp2-2024_tp2 -Dsonar.organization=devops-tp2-2024 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file backend/simple-api-student-main
```

Now as soon as a github push is initiated, once the unit and integration tests complete the performance will be sent directmenet to sonar which we ssitura notamenet the technical debt


## TP3 - Hugo MONTAGNON

## Inventory Setup

By default, Ansible's inventory is stored in `/etc/ansible/hosts`. However, for this project, we've created a project-specific inventory file located at `TP/ansible/inventories/setup.yml`.

### Content

The inventory file `setup.yml` is structured as follows:

```yaml
all:
  vars:
    ansible_user: centos
    ansible_ssh_private_key_file: /home/hmontagnon/.ssh/id_rsa
  children:
    prod:
      hosts: hugo.montagnon.takima.cloud
```

`ansible_user`: Specifies the user to use for SSH connections.

`ansible_ssh_private_key_file`: Specifies the path to the SSH private key file. (path relate to ubuntu)

`prod` : Group containing the hosts to be managed by Ansible.


### Testing Inventory

To verify that Ansible can communicate with your hosts, use the ping module:

```sh
ansible all -i inventories/setup.yml -m ping
```

### Retrieving OS Distribution

To get information about the OS distribution of your hosts, use the setup module with a filter:

```sh
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
```

### Removing Apache HTTP Server

To remove the Apache HTTP server:

```sh
ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become
```
Running this command again will not change the server state if httpd is already removed, demonstrating Ansible's idempotency.

## Playbooks

### First Playbook

Create a file named ansible/playbook.yml in the main directory with the following content:

```yml
- hosts: all
  gather_facts: false
  become: true
```

Run the playbook using the following command:

```sh
ansible-playbook -i inventories/setup.yml playbook.yml
```

Before we can check our playbook for syntax errors before running it:

```sh
ansible-playbook -i inventories/setup.yml playbook.yml --syntax-check
```

The `First Playbook` is mainly used to test the connection between Ansible and the hosts defined in your inventory. This is an essential first step to ensure that SSH configuration and other connection parameters are correct, before proceeding with more complex tasks.

### Advanced Playbook

Let's create a more advanced playbook to install Docker on your server
Create a file named docker-install.yml in the directory ansible/ with the following content:

```yml
- hosts: all
  gather_facts: false
  become: true

  tasks:
    - name: Install device-mapper-persistent-data
      yum:
        name: device-mapper-persistent-data
        state: latest

    - name: Install lvm2
      yum:
        name: lvm2
        state: latest

    - name: Add Docker repository
      command:
        cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

    - name: Install Docker
      yum:
        name: docker-ce
        state: present

    - name: Install Python3
      yum:
        name: python3
        state: present

    - name: Install Docker with Python3
      pip:
        name: docker
        executable: pip3
      vars:
        ansible_python_interpreter: /usr/bin/python3

    - name: Ensure Docker is running
      service:
        name: docker
        state: started
        tags: docker
```
Run the advanced playbook using the following command:

```sh
ansible-playbook -i inventories/setup.yml docker-install.yml
```

`docker-install.yml` is an Ansible playbook designed to automate the installation of Docker on servers managed by Ansible. It ensures that Docker and its dependencies are correctly installed and configured on target hosts. 

### Using Roles

Roles in Ansible are a method of structuring playbooks and reusing code. They allow you to group related tasks, handlers, variables, files, templates and other resources into logical sets. 

So let's create a Docker role and move the installation task there:

```sh 
ansible-galaxy init roles/docker
```

In this project we will only keep the tasks, the others have been suppressed as useless.

### Move Tasks to the Role

Edit `roles/docker/tasks/main.yml`

```yml
- name: Install device-mapper-persistent-data
  yum:
    name: device-mapper-persistent-data
    state: latest

- name: Install lvm2
  yum:
    name: lvm2
    state: latest

- name: Add Docker repository
  command:
    cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

- name: Install Docker
  yum:
    name: docker-ce
    state: present

- name: Install Python3
  yum:
    name: python3
    state: present

- name: Install Docker with Python3
  pip:
    name: docker
    executable: pip3
  vars:
    ansible_python_interpreter: /usr/bin/python3

- name: Ensure Docker is running
  service:
    name: docker
    state: started
    tags: docker
```
This playbook automates the installation and setup of Docker on a CentOS system. It first installs necessary dependencies (device-mapper-persistent-data and lvm2), adds the Docker repository, installs Docker, and ensures that Docker is running. 

It also installs Python3 and the Docker Python module, allowing for further Docker management through Python scripts or Ansible modules that require Python3.

### Use the Role in the Playbook

Edit playbook.yml to include the Docker role:

```yml
- hosts: all
  gather_facts: false
  become: true

  roles:
    - docker
```

Run the playbook using the following command:

```sh
ansible-playbook -i inventories/setup.yml playbook.yml
```

## Deploy your App

 ***We apply the same role principle to download images and launch docker containers directly from a dockerhub account.***

### **Tree:**

    roles/
    └── docker/
        ├── tasks/
        │   └── main.yml
        network/
        ├── tasks/
        │   └── main.yml
        database/
        ├── tasks/
        │   └── main.yml
        backend/
        ├── tasks/
        │   └── main.yml
        proxy/
        ├── tasks/
        │   └── main.yml

#### To install docker

Edit `roles/docker/tasks/main.yml` :

```yml
---
# tasks file for roles/docker
# Advanced Playbook for Docker Installation

# Install Docker


- name: Install device-mapper-persistent-data
  yum:
    name: device-mapper-persistent-data
    state: latest

- name: Install lvm2
  yum:
    name: lvm2
    state: latest

- name: Add Docker repository
  command: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

- name: Install Docker
  yum:
    name: docker-ce
    state: present

- name: Install Python 3
  yum:
    name: python3
    state: present

- name: Install docker package for Python 3
  pip:
    name: docker
    executable: pip3
  vars:
    ansible_python_interpreter: /usr/bin/python3

- name: Ensure Docker is running
  service: name=docker state=started
  tags: docker

- name: Login to Docker Hub
  docker_login:
    username: hmtgn
    password: Cailloux123?
    reauthorize: yes
  vars:
    ansible_python_interpreter: /usr/bin/python3
```

#### To create network

Edit `roles/network/tasks/main.yml` :

```yml
- name: Create application network
  docker_network:
    name: app-network
    state: present
  vars:
    ansible_python_interpreter: /usr/bin/python3
```

#### To launch database

Edit `roles/database/tasks/main.yml` :

```yml
- name: Pull the BDD image
  docker_image:
    name: hmtgn/tp1-database
    tag: latest
    source: pull
  vars:
      ansible_python_interpreter: /usr/bin/python3

- name: Run DB
  docker_container:
    state: started
    name: database
    image: hmtgn/tp1-database
    networks: 
      - name: "app-network"
  vars:
      ansible_python_interpreter: /usr/bin/python3
```
#### To launch app

Edit `roles/backend/tasks/main.yml` :

```yml
- name: Pull the API Image
  docker_image:
    name: hmtgn/tp1-backend
    tag: latest
    source: pull
  vars:
      ansible_python_interpreter: /usr/bin/python3

- name: Run API
  docker_container:
    name: backend
    image: hmtgn/tp1-backend
    networks: 
      - name: "app-network"
    state: started
  vars:
      ansible_python_interpreter: /usr/bin/python3
```

#### To launch proxy

Edit `roles/proxy/tasks/main.yml` :

```yml
- name: Pull the proxy container
  docker_image:
    name: hmtgn/tp1-httpd
    tag: latest
    source: pull
  vars:
      ansible_python_interpreter: /usr/bin/python3
 
- name: Run the proxy container
  docker_container:
    name: httpd
    image: hmtgn/tp1-httpd
    ports:
      - "80:80"
    networks:
      - name: "app-network"
  vars:
      ansible_python_interpreter: /usr/bin/python3
```

#### Finally, the playbook must be modified

```yml
- hosts: all
  gather_facts: false
  become: true

  roles:
    - docker
    - network
    - database
    - backend
    - proxy
```

## Continuous Deployment

The aim now is to automate deployment of the application via github action and a new `deploy.yml` file like this:

```yml
name: Deploy
on:
  workflow_run:
    workflows: ["CI devops 2024"]
    types:
      - completed

jobs:
  deploy:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v2.5.0

      - name: Setup SSH and known hosts
        uses: webfactory/ssh-agent@v0.5.4
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Install Ansible
        run: |
          sudo apt-get update
          sudo apt-get install -y ansible

      - name: Disable host key checking
        run: echo "ANSIBLE_HOST_KEY_CHECKING=False" >> $GITHUB_ENV

      - name: Deploy to production server
        run: |
          ansible-playbook -i ansible/inventories/setup.yml ansible/playbook.yml
```

The latter will be launched after the first main.yml file on ggithub action and will repeat all the steps explained above. 

***/!\ However, the content of the ssh key must be added to the secret variables so that the connection can be established.*** 



## Author

This project was developed by [Hugo Montagnon](https://github.com/hmtgn).