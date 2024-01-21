# Cloud-Native System Monitoring App in Flask and Plotly with AWS (EKS, EKR), Docker, and Kubernetes

1. **AWS Access Keys** are credentials used to authenticate to the AWS APIs. Any time you execute the aws command line tools, or use any kind of tool or script that interacts with AWS, access keys are what you use to identify yourself to AWS. They’re tied to an IAM principal — either an IAM user or an IAM role. Access keys are broken down into three primary components:

- Access Key ID - roughly equivalent to a username, this is the unique identifier for the set of access keys you’re using. This can be found in CloudTrail logs when using long-lived access keys.

- Secret Access Key - the secret component of any set of credentials, these are used to sign requests to the AWS API. Consider them equivalent to a password, and protect them accordingly.

We will set up our AWS cli by creating an access key and logging in via the terminal by using the aws configure command. And now we have programmatic access to aws.

2.  We created our app with the help of psutil module to get real-time system information and to display it in the form of guages, we used plotly module in python.

3. Now we will containerize our app using docker. So we first created a dockerfile. \
   It begins by using the official Python 3.9 image based on the Debian "bullseye" (Debian 11) slim variant, optimizing image size. The working directory inside the container is set to /app, and the requirements.txt file, containing necessary Python dependencies, is copied into this directory.\
   The Docker image installs the dependencies using pip3 install, with the --no-cache-dir flag to prevent caching of package files,  reducing the size of the Docker image. The entire local directory, assumed to contain the application code, is then copied into the container's /app directory.\
   An environment variable FLASK\_RUN\_HOST is set to 0.0.0.0, allowing external access to the Flask application. The Dockerfile informs Docker that the application will listen on port 5000 (EXPOSE 5000), and the default command (CMD \[ "flask", "run" ]) specifies running the Flask development server when the container starts.\
   This  command builds our docker image using the dockerfile.\
   **docker build -t my-flask-app .**

4. This is what the web-app looks like with live-updates mode -\

   ![](https://lh7-us.googleusercontent.com/E7UAYHAO3jGf-82v4bC8ShBMqkuhQpGzNFgOKmCvovmA-5bgiS0r1CnLYrh5oMz82KYno4vqhPeyQB0wV1VMhnekFaP3my-t40gO7A8oJsArDwrCRN5uXy7fQG-9x6uuJRs3qzVwRUvBrTzl0U5Rd64)\
   (this is the real-time updates guage, we have pushed the static guage to docker, which has get requests to backend every second)\
   We will push this app to the **Amazon Elastic Container Registry** but in **VScode** using the **boto3** module (the AWS SDK module for Python).****

5. Starting from this- \
   <https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ecr.html#client>\
   We create a repository on the ECR.  An ECR **repository (or repo)** is a location where you can store, manage, and deploy Docker container images. These repositories can be used to store Docker images for use with services like Amazon Elastic Kubernetes Service (EKS), Amazon Elastic Container Service (ECS), or any container orchestration platform that supports Docker containers.

|                                                                                                                                                          |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ecr\_client = boto3.client('ecr') repository\_name = "my-cloud-native-repo" response = ecr\_client.create\_repository(repositoryName = repository\_name) |

6. Now we will push our image into this repo. We view the push commands inside our created repo in aws. We follow the steps/commands it says.\
   ->  retrieve an authentication token from AWS ECR, to authenticate the Docker client with the\
         ECR registry\

   &#x20;->  build a Docker image from the source code located in the current directory (.)   \
   &#x20;->  tag the Docker image with the repository URI to which it will be pushed \
   &#x20;->  push the tagged Docker image to the specified ECR repository.\
   The 1st step is crucial for securing the push operation to the ECR repository.

7. Now we create a kubernetes cluster on the **Amazon Elastic Kubernetes Service**. By now we have deployed our image on the ECR repo. We will use the EKS to deploy our containers in the form of kubernetes pod. When our cluster becomes active after creation, we add a node group to it. Which will have 2 nodes inside it.

8. We move to creating kubernetes **deployment** and **service** using the kubernetes python client.\
   
    In Kubernetes, a **"Deployment"** is an object that provides declarative updates to applications. It allows you to describe the desired state for your application, such as the number of replicas, which container images to use, and how to update them. The Deployment controller then continuously works to ensure that the current state of the deployed application matches the desired state.\
   
    a "**Service**" is an abstract way to expose an application running on a set of Pods as a network service. It provides a stable endpoint (IP address and port) that other applications can use to access the deployed pods. Services play a crucial role in enabling communication between different parts of your application or between different applications within a Kubernetes cluster.\
   
    ↪️ Side-note - with the eks.py, where i had to put the Image URI, I accidently put the wrong one and executed the file. This change could be reversed back by deleting the service, pod and the deployment, the file had created

9. Finally, after successfully running eks.py we have our pods up and running. (it required 4 nodes of t2.micro)\
   &#x20;kubectl get pods -n default\

   ![](https://lh7-us.googleusercontent.com/8ls-FNURxVOdjKUeqHytMWU5dYJB_TZB4k4uYHdJPKeRc7g0MTdqWeW023ePR1DVtzVEMcsA7FpYyTGu3tv8PBnbM6TKv5SEWEwfnlIhYaKi6-oRi42TAzcv58Uw9azmxgFhbLHEEXBMzWxdvI266Uw)\
   
   And on\
   &#x20;kubectl port-forward svc/my-flask-service 5000:5000\
   we have our service running on port 5000 in the container, which by exposing/forwarding the port to localhost, we can use the pod on our local machine as well-\

   ![](https://lh7-us.googleusercontent.com/YtoOG6v4A39gxLDwm47VA6iIyKTzHKtIFyl7hpM9CJzYhvH1emITbz-ZH--1Wx7xSmGEw8fndag4zFGtMiJ0bplthBBA5I2k7MkjoKjbX-5gTkri-RTWfnsq-jt3w2PR_ReyrLHQsE-bfrOZn07oewY)&#x20;

10. Clean-up (to not get charged) -\
    First we will delete our node-group\
    Then after that we will delete our cluster as well,\
    And finally we will delete our ECR repository too.
