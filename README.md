# CI
# Assignment 1

# Phase I:

## - Create a new github public repo from [repo]

1. Used `git clone https://github.com/kubernetes/examples.git` to clone the repository on local machine
2. Removed unnecessary files from Local Repo
3. Added local git repo using `git init` command 
4. Added the Remote Repository using `git remote add remote-name remote-url`
5. Added required files to staging area using `git add .`
6. Committed changes using `git commit -m ".."`
7. Pushed files to remote repo using `git push origin main` 

## - Create a container image using Github Action workflow for above project.

1. **Create Docker Credentials in GitHub Secrets**:
Before you set up the GitHub Actions workflow, ensure that you have the necessary credentials for the container registry where you want to push your Docker image. These credentials should be stored as secrets in your GitHub repository.
    
    Go to your GitHub repository, navigate to "Settings" > "Secrets" > "New repository secret" and create the following secrets:
    
    - **`DOCKER_USERNAME`**: Your Docker Hub username (or username for the container registry you are using).
    - **`DOCKER_PASSWORD`**: Your Docker Hub password or access token (or password for the container registry you are using).
2. **Create a GitHub Actions Workflow**:
In your repository, create a **`.github/workflows`** directory if it doesn't exist. Inside the **`workflows`** directory, create a new YAML file, e.g., **`build-image.yml`**.

### Problem with Given Repo

Code in `Dockerfile` 
`go get [github.com/xyproto/simpleredis](http://github.com/xyproto/simpleredis/)` giving error because it is updated to [`github.com/xyproto/simpleredis/](http://github.com/xyproto/simpleredis/)v2` 

Also Code in `main.go` has to be updated to have [`github.com/xyproto/simpleredis/](http://github.com/xyproto/simpleredis/)v2` 

![Untitled](https://github.com/shivam1750/Assignment/blob/main/images/build%20.png)

## - Push that image to docker hub.

Explanation of YAML code in `.github/workflows/build-push-image.yml` which is used to build and push image to docker hub

1. The **`on`** section specifies that the workflow should be triggered on any push to the **`main`** branch.
2. The **`jobs`** section defines a single job named **`build`** that runs on an **`ubuntu-latest`** runner.
3. Inside the **`build`** job, there are four steps:
    - **`Checkout repository`**: This step uses the **`actions/checkout@v2`** action to clone your repository into the runner environment.
    - **`Log in to Docker Hub`**: This step runs the **`docker login`** command, using the Docker Hub username and password (or access token) stored as secrets in your GitHub repository. This ensures that Docker can authenticate with Docker Hub and push the image.
    - **`Build Docker image`**: This step runs the **`docker build`** command to build the Docker image using the Dockerfile located in the repository's root directory. The resulting image is tagged as **`ajinkyak423/assignment:latest`**, where **`ajinkyak423`** is your Docker Hub username and **`assignment`** is the desired name for your Docker image.
    - **`Push Docker image to Docker Hub`**: This step runs the **`docker push`** command to push the built image (**`ajinkyak423/assignment:latest`**) to Docker Hub, making it accessible to others and usable for deployments.

With this configuration, GitHub Actions will automatically trigger the workflow whenever you push changes to the **`main`** branch, and it will build and push the Docker image to Docker Hub, making it available for use.
![Untitled](https://github.com/ajinkyak423/Assignment/blob/2aa81bab028415322ac2203d8baddfbf335b5f78/images/docker%20hub.png)

# Phase II

## - Deploy container image built in above step(Phase I) on local/any Kubernetes cluster.

1. Using Minikube as local K8s cluster to deploy Containerized application built in phase-I
2. Started Minikube using `minikube start --driver=hyperv`
3. Checked status of minikube using `minikube status`
4. Created `deployment.yaml` for deploying app on minikube
    
    Code Explanation:
    
    1. **`kind: Deployment`**
        - The **`kind`** field specifies the type of Kubernetes object being defined, which is a Deployment in this case.
    2. **`metadata:`**
        - This section contains metadata about the Deployment, such as the name (**`assignment-deployment`**).
    3. **`spec:`**
        - The **`spec`** section specifies the desired state of the Deployment.
    4. **`replicas: 1`**
        - This line indicates that the desired number of replicas (pods) for this Deployment is 1. The Deployment will ensure that one replica of the application is always running.
    5. **`selector:`**
        - The **`selector`** field is used to match the pods controlled by this Deployment. In this case, it uses the label **`app: assignment`**.
    6. **`template:`**
        - The **`template`** section defines the pod template that will be used to create the pods managed by this Deployment.
    7. **`metadata:`**
        - This section contains metadata about the pod template.
    8. **`labels:`**
        - The **`labels`** field specifies labels for the pods created from this template. In this case, it applies the label **`app: assignment`** to the pods.
    9. **`spec:`**
        - The **`spec`** section within the **`template`** defines the desired state of the pod created from this template.
    10. **`containers:`**
        - This section specifies the containers running in the pod.
    11. **`name: assignment-container`**
        - This line sets the name of the container to **`assignment-container`**.
    12. **`image: ajinkyak423/assignment:latest`**
        - This line specifies the Docker image to be used for the container. The image **`ajinkyak423/assignment:latest`** will be pulled and used to create the pod.
    13. **`ports:`**
        - This section defines the ports to be exposed on the container.
    14. **`containerPort: 3000`**
        - This line specifies that the container listens on port **`3000`**. It doesn't expose the port to the outside world; it's just a declaration of what the container is listening on internally.

5. Created `service.yaml` file for creating service for application of `Nodeport` type
Code Explanation:
    
    Service named **`assignment-service`** that exposes the pods labeled with **`app: assignment`**. The Service uses **`NodePort`** type to expose the pods to the outside world. It maps port **`3000`** of the Service to port **`3000`** of the pods and exposes it on a randomly selected node port **`30080`**. With this configuration, you should be able to access your application externally using the node's IP address and the node port **`30080`**.
    
6. Obtained Minikube Ip using `minikube ip`
7. Accessed application using **`<Minikube-IP>:30080`**
![Untitled](https://github.com/shivam1750/Assignment/blob/8dfb055172bd8294379fbb819dedc8d3308a3db0/images/minikube%20setup.png)

![Untitled](https://github.com/shivam1750/Assignment/blob/8dfb055172bd8294379fbb819dedc8d3308a3db0/images/minikube%20deploymane.png)

# Phase III

## - Prevent merging anything in main branch without review.

1. Under "Branch protection rules", `main` is added to target as branch
2. Enabled the "Require pull request reviews before merging" option.
3. Enabled other options like "Require status checks to pass before merging" and "Require signed commits" for additional protection.

## - Build container image only when one of the below conditions is true:

1.  When PR get merged in main/master branch from any other branch.

    
    ![Untitled](https://github.com/shivam1750/Assignment/blob/8dfb055172bd8294379fbb819dedc8d3308a3db0/images/pull%20request%20from%20test%20branch%20to%20main.png)
    
    ![Untitled](https://github.com/shivam1750/Assignment/blob/8dfb055172bd8294379fbb819dedc8d3308a3db0/images/confirming%20merge%20.png)
    
    ![Untitled](https://github.com/shivam1750/Assignment/blob/8dfb055172bd8294379fbb819dedc8d3308a3db0/images/pull%20request%20merged.jpg)
    
2.  When commit message contains BUILD_CONTAINER_IMAGE string
   a. Added script in `Phasee_III-Condition-build.yaml` file to do so 

```yaml
- name: Check for commit message
        id: check_commit_message
        run: |
          if grep -q "BUILD_CONTAINER_IMAGE" <<< "${{ github.event.head_commit.message }}"; then
            echo "commit_message=true" >> $GITHUB_ENV
          else
            echo "commit_message=false" >> $GITHUB_ENV
          fi

      - name: Log in to Docker Hub
        run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker image
        if: env.commit_message == 'true' || (github.event_name == 'pull_request' && github.event.pull_request.merged)
        run: docker build -t ajinkyak423/assignment:latest .

      - name: Push Docker image to Docker Hub
        if: env.commit_message == 'true' || (github.event_name == 'pull_request' && github.event.pull_request.merged)
        run: docker push ajinkyak423/assignment:latest
```

first check if the commit message contains **`BUILD_CONTAINER_IMAGE`**, and we set the **`commit_message`** environment variable accordingly. Then, in the **`build`** job, we use the **`if`** condition to check the value of **`commit_message`** and whether the event is a merged pull request. If either condition is true, the Docker image will be built and pushed. Otherwise, the build steps will be skipped.

![Untitled](https://github.com/ajinkyak423/Assignment/blob/8c80bdf6734749de3db0e54e03235bf857cc9844/images/no%20build.png)

![Untitled](https://github.com/ajinkyak423/Assignment/blob/8c80bdf6734749de3db0e54e03235bf857cc9844/images/condition%20build%20success.png)

![Untitled](https://github.com/ajinkyak423/Assignment/blob/8c57759d19d3ac07bce0d1019574ad9dd7714f16/images/build%20after%20pull%20req%20merged.png)
