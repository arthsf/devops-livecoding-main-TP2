# **Docker**
# _TP1 - Discovering Docker_

> The final goal of this TP is to have a 3-tier web API running.

## 1-1 Document your database container essentials: commands and Dockerfile.
- **FROM postgres:latest**: This command specifies the base image for your container. In this case, it uses the latest official PostgreSQL image from Docker Hub.
- **ENV**: These commands set environment variables in the container. They are essential for configuring your PostgreSQL instance:
    - **POSTGRES_USER**: Specifies the PostgreSQL username.
    - **POSTGRES_PASSWORD**: Sets the password for the PostgreSQL user.
    - **POSTGRES_DB**: Defines the name of the PostgreSQL database to be created.
- **EXPOSE 5432**: This command exposes the default PostgreSQL port (5432) to allow communication with the PostgreSQL database from outside the container.
- *COPY*: This command copies files from the host machine to the container. In this case, it's copying an initialization SQL script (init.sql) to the /docker-entrypoint-initdb.d/ directory within the container. Initialization scripts placed in this directory will be executed when the PostgreSQL container is first started, allowing you to set up the database schema or perform other initial configurations.

## 1-2 Why do we need a multistage build? And explain each step of this dockerfile.
A multistage build is useful in this Dockerfile for a Java simple API because it separates the build environment from the runtime environment, resulting in a smaller and more secure final image.
- The Dockerfile has two stages: a build stage and a runtime stage.
- In the build stage, it uses an Amazon Corretto-based Maven image to build the Java application. It sets up the working directory, copies the project files, and runs Maven to package the application.
- In the runtime stage, it uses the Amazon Corretto image. It sets up the working directory and copies the JAR file built in the previous stage into the runtime image.
- Finally, it specifies the entry point to run the Java application using java -jar

## 1-3 Why do we need a reverse proxy?
A reverse proxy serves several purposes, including enhancing security, optimizing performance, load balancing, and centralization. It achieves these benefits by hiding the internal structure of a network, preventing direct access to backend services, caching content to reduce the load on backend servers, distributing incoming requests to balance the workload across multiple backend servers, and providing a single access point for clients while managing SSL/TLS, freeing up backend applications from this responsibility.

## 1-4 Why is docker-compose so important?
Docker-compose is crucial because it simplifies the definition and execution of multiple Docker containers as a unified set. With just one docker-compose.yml file, you can define services, networks, and volumes and start the entire application with a single command (docker-compose up). This streamlines the management, updating, and scaling of interdependent services in development, testing, and production environments.

## 1-5 Why do we put our images into an online repo?
We put our images into an online repository to facilitate collaboration, distribution, and deployment. Storing images in an online repository like Docker Hub allows other developers, systems, or services to easily download and run these images without the need to build them locally. This simplifies the deployment of applications in various environments or on multiple machines, as images can be quickly retrieved and executed from the repository. Additionally, it serves as a form of backup for your Docker images.

# _TP2 - GitHub Actions_

## 2-1 What are testcontainers
Testcontainers are Java libraries that enable the execution of Docker containers during tests. In the described context, they are used to launch a PostgreSQL container during test execution, providing a database in which tests can insert or extract data, ensuring that the application's database interaction functionalities work as expected. This approach helps in testing database-related features more effectively and in an isolated environment.

## 2-2 Document your Github Actions configurations
GitHub Actions is a service on GitHub that automates tasks like code building and testing. In this context, it's configured to:
- Trigger on code pushes and pull requests.
- Run on an Ubuntu 22.04 virtual machine.
- Fetch the source code.
- Set up JDK 17.
- Build and test the application with Maven.
- If tests pass, build and push Docker images to Docker Hub.

Secure variables store Docker Hub credentials to keep them safe. This configuration is described in a main.yml file in the .github/workflows directory of the repository, specifying each step in the workflow using predefined GitHub actions and custom commands.

# _TP3 - Ansible_

## 3-1 Document your inventory and base commands
The inventory setup is documented in the setup.yml file located within the inventories directory. Within this inventory:
- ansible_user: Signifies the user utilized for connecting to the remote machine, which, in our case, is 'centos.'
- ansible_ssh_private_key_file: Points to the path of our SSH private key, specifically '~/.ssh/id_rsa,' following its relocation.

Within the 'prod' group, we've delineated our server, denoted as 'arthus.frappe.takima.cloud.'
Here are the fundamental commands that were executed:
1. Ping: We conducted a connectivity check to our server by executing the command `ansible all -i inventories/setup.yml -m ping.` The successful connection is indicated by the response "pong."
2. Information Gathering: To fetch specific details about the server's distribution, we employed the command `ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*." It was determined that our server operates on CentOS 7.9.
3. Apache httpd Removal: We made an attempt to uninstall Apache httpd using the command `ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become.` The result indicated that Apache httpd was not installed.

## 3-2 Document your playbook
In this playbook, we have defined a series of tasks to install Docker on our CentOS-based server. Here is a summary of the steps followed:
1. Package Cleanup: We begin by cleaning up packages using the "yum clean" command.
2. Installation of device-mapper-persistent-data: This package is essential for Docker to work correctly on CentOS.
3. Installation of lvm2: Another necessary package for Docker, and we ensure that it is installed and up to date.
4. Adding the Docker Repository: We add the official Docker repository for CentOS to be able to install Docker directly from this repository.
5. Docker Installation: After adding the repository, we proceed to install Docker.
6. Starting Docker: Finally, we ensure that the Docker service is started and functioning correctly on the server.

Regarding the "docker_container" tasks configuration:

We have defined a "docker_container" task in the "proxy" role to deploy an HTTPD container. In this task, we specified the container name as "httpd" and used the image "arthsf/tp-devops-httpd:latest" We also exposed port 80 of the container and linked it to port 80 on the host. Additionally, this container is attached to the network named "my-network," which we created earlier to allow communication between our different containers.

