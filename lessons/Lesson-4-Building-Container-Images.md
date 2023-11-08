# Lesson 4: Building Container Images

- [Lesson 4: Building Container Images](#lesson-4-building-container-images)
    - [4.1 Understanding Image Formats](#41-understanding-image-formats)
      - [Understanding Images](#understanding-images)
      - [Getting Container Images](#getting-container-images)
      - [Fetching Images from Registries](#fetching-images-from-registries)
      - [Demo: Managing Container Images](#demo-managing-container-images)
    - [4.2 Using Dockerfile](#42-using-dockerfile)
      - [Understanding Dockerfile](#understanding-dockerfile)
      - [Demo: Using a Dockerfile](#demo-using-a-dockerfile)
    - [4.3 Creating a GitOps Container Image](#43-creating-a-gitops-container-image)
      - [Creating a GitOps Container Image](#creating-a-gitops-container-image)
    - [4.4 Using Webhooks to Automate Container Image Updates](#44-using-webhooks-to-automate-container-image-updates)
      - [Docker steps](#docker-steps)
      - [Update to test build](#update-to-test-build)

### 4.1 Understanding Image Formats

#### Understanding Images

- A container is a running instance of an image.
- The image contains application code, language runtime, and libraries.
- External libraries, such as libc, are tuypically provided by the host operating system, but in container are included in the image.
- While starting a container the image is copied to a directory on the host where it runs.
- Container images are highly compatible, and either defined in Docker or in OCI format.

#### Getting Container Images

- Containers are normally fetched from registries.
- The Docker registry at https://hub.docker.com contains many images and is commonly used.
- Other public image registries are available as well.
- Alternatively, private registries can easily be created.
- Use Dockerfile/Containerfile to create custom images.

#### Fetching Images from Registries

- Kubernetes by default fetches images from Docker hub and the Google container registry.
- To avoid confusion, it is wise to use a fully qualified container image name: **docker.io/library/nginx** instead of nginx.

#### Demo: Managing Container Images

```bash
# Explore https://hub.docker.com
docker search mariadb # will search for the mariadb image.
docker pull mariadb
docker images
docker inspect mariadb
docker image history mariadb
docker image rm mariadb
```

### 4.2 Using Dockerfile

#### Understanding Dockerfile

- Dockerfile is a way to automate container builds.
- In Red Hat environments, "Containerile" is used instead of "Dockerfile".
- It contains all instructions required to build a container image.
- Use **docker build . -t docker.io/yourname/myapp:1.0** to build the container image based on the Dockerfile in the current directory and tag it for storage on docker.io
- Images will be stored on your local system, and provide the right tag to facilitate uploading it to Docker hub.
- Tip: many images on hub.docker.com have a link to Dockerfile. Read it to understand how an image is built using Dockerfile!

#### Demo: Using a Dockerfile

```bash
# Docker demo file in labs/dockerfiledemo/Dockerfile.
# Build docker image.
cd labs/dockerfiledemo/
docker build -t docker.io/anilitblr/nmap:1.0 .
docker build --no-cache -t docker.io/anilitblr/nmap:1.0 . # to ensure the complete procedure is performed again if you need to run it again.
docker run nmap # to run it.
docker push docker.io/anilitblr/nmap:1.0 # To push it to Docker hub.
```

### 4.3 Creating a GitOps Container Image

#### Creating a GitOps Container Image

- Container images are developed to be small.
- As a result, it may be hard to find an image that has all the tools that you may need.
- So let's create a GitOps container image.
- The example is in the course Git repository at **labs/gitopstools**

```bash
cd labs/gitopstools
cat Dockerfile
docker build -t gitops .
docker images
docker login
docker tag gitops your_docker_username_here/gitops
docker push your_docker_username_here/gitops
```

### 4.4 Using Webhooks to Automate Container Image Updates

- Access https://github.com
- Click **Create Repository**
- Enter the name of the new Git repo; e.g, **gitopshook** and set to **Public**
- Click **Create Repository**
- Check **Settings > Webhooks** Don't change anything, but check again later.
- On a Linux console, create the local repository.
```bash
mkdir gitopshook
cd gitopstools
echo "Hello" >> README.md

cat > Dockerfile <<EOF
    FROM busybox
    CMD echo "Hello world!"
    EOF

cat Dockerfile
git init
git add *
git commit -m 'initial commit'
git branch -M main
git remote add origin https://github.com/yourname/gitopshook
git push -u origin main
```

#### Docker steps

- Still from hub.docker.com: Add a **Build Rule** that sets the following.
  - **Source Type**: Branch
  - **Source**: main
  - **Docker Tag**: latest
  - **Build Context**: /
- Click **Build and Run**
- Check **Builds > Build Activity** to see progress (give it 3 minutes)
- Once the build is completed successfully, from a terminal use **docker pull yourname/gitopshook:latest** to pull this latest image (may take a minute to synchronize)
- On GitHub, check **Settings > Webhooks** for your repo - settings should be automatically added.

#### Update to test build

- From the Git repo on the Linux console: edit the Dockerfile and add the following line: MAINTAINER yourname <your@email.com>
```bash
git status
git add *
git commit -m 'minor update'
git push -u origin main
```
- From Docker hub: Check your **repository > Builds > Build Activity**. You will see that a new automatic build has been triggered (Should be fast.)
```bash
docker pull yourname/imagename
```
