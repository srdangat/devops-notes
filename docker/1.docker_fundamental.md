### What is Docker?

- It’s an instance of an image that is ready to execute code with all its necessary libraries,configurations and files.

### Why we need?

- You can build locally,deploy to the cloud and run anywhere on any server.
- Shares the host kernel,making them much more efficient in therms of system resources than vitual machine.
- Makes collaboration easier ,same environment for everyone
- No version mismatch issues.


### Containers vs Virtual Machines

| Feature | Virtual Machines (VMs) | Containers |
|----------|------------------------|------------|
| **Virtualization Level** | Hardware-level virtualization | OS-level virtualization |
| **Architecture** | Includes full guest OS + hypervisor | Shares host OS kernel |
| **Size** | Large (GBs) | Small (MBs) |
| **Startup Time** | Slow (minutes) | Fast (seconds) |
| **Performance** | Slower due to OS overhead | Faster, lightweight |
| **Isolation** | Strong (separate OS per VM) | Process-level isolation |
| **Resource Usage** | High CPU, RAM, Storage usage | Efficient resource usage |
| **Portability** | Less portable | Highly portable |
| **Management** | Complex (manage full OS) | Simple (manage app + dependencies) |
| **Best For** | Legacy apps, multiple OS environments | Microservices, CI/CD, cloud-native apps |



### Docker architecture


![image](1770296705177.gif)

---

### Docker Client

### What it is
The Docker client is the command-line interface (CLI) used to interact with Docker. It acts as the command center.

### How it works
You type commands in the Docker client, and it sends those requests to the Docker daemon, which performs the actual work.

### Example Commands
- `docker build`
- `docker run`
- `docker pull`
- `docker push`

---

### Docker Daemon

### What it is
The Docker daemon (`dockerd`) is the background service that manages Docker objects such as images, containers, networks, and volumes.

### How it works
The daemon:
- Listens for Docker API requests from the Docker client
- Builds images
- Runs and manages containers
- Handles networking and storage


---

### Docker Hub

### What it is
Docker Hub is a cloud-based public registry for Docker images.

### How it works
It works like an app store for container images. 

You can:
- **Pull** images created by others
- **Push** your own images

### Usage
When you need an image to create a container, you can pull it from Docker Hub.

---

### Docker Registry

### What it is
A Docker registry is a system that stores and distributes Docker images. Docker Hub is the most popular public registry,but you can also create private registries.

### How it works
Registries:
- Store Docker images
- Allow users to pull images
- Allow users to push images

Private registries are commonly used by companies to securely store internal application images.


### Install Docker

- https://docs.docker.com/engine/install/