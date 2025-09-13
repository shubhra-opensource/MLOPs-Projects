# Introduction to Docker and Containerization

Containers are **lightweight**, resource-efficient, and portable. They share a single host operating system (OS) with other containers—sometimes hundreds or thousands—while isolating software code from the underlying environment. This allows developers to build applications on one host OS (such as Linux) and deploy them reliably on other OSes (like Windows) without worrying about configuration issues.

**Docker** packages, provisions, and runs containers by bundling an application and all its dependencies, libraries, and configuration files together. Unlike virtual machines (VMs), which duplicate an entire OS, Docker leverages **resource isolation in the OS kernel** to efficiently run multiple containers on the same host. Docker images contain everything needed for reproducible execution, so containers can move between Docker environments seamlessly.

***

## Why Use Docker?

- **Reproducibility:** Ensures a consistent environment for development, testing, and deployment.
- **Isolation:** Each container is isolated from others and the host.
- **Portability:** Run anywhere Docker is supported—on your laptop, server, or the cloud.

***

## The Moby Project

Docker builds upon the open-source **Moby** project, which provides:

- **A library of backend components** for container functionality (image building, storage, logging).
- **A framework and toolchain** for assembling and testing containers across OSes and cloud environments.
- **Reference assemblies and examples**—the open-source core for Docker.

Explore the Moby project here:  
[https://github.com/moby/moby](https://github.com/moby/moby)

***

## Docker Installation

- **macOS and Windows:** [Install Docker Engine](https://docs.docker.com/engine/install/)
- **Linux (e.g., Ubuntu):** [Install Docker on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- **Windows Subsystem for Linux (WSL) Recommended:** [WSL Setup Guide](https://docs.docker.com/desktop/windows/wsl/)

### Fix Permissions on Linux

```bash
sudo usermod -aG docker $USER
```

***

## Quickstart: Hello World

Test your Docker installation:

```bash
docker run hello-world
```

Try out Docker in your browser (no installation required):  
[Play With Docker](https://labs.play-with-docker.com/)

***

## Running an Ubuntu Container

```bash
docker run -it ubuntu bash
```
Where `-it` means *interactive terminal*.

***

## Docker Architecture Overview

Docker uses a **client-server architecture**:

- The **client** sends commands to the **daemon**, which builds, runs, and manages containers.
- Communication uses a REST API over UNIX sockets or network interfaces.
- **Docker Compose** helps you manage multi-container applications.

***

## Key Docker Commands

| Description                           | Command Example                                           |
| -------------------------------------- | -------------------------------------------------------- |
| List all containers (incl. exited)     | `docker ps -a`                                            |
| List Docker images                     | `docker images`                                           |
| Start a container                      | `docker start <container_name>`                           |
| Inspect a container                    | `docker inspect <container_name>`                         |
| Enter a running container              | `docker exec -it <container_name> bash`                   |

See the [Docker CLI reference](https://docs.docker.com/engine/reference/commandline/docker/) for full details.

***


## Understanding the Dockerfile

Example `Dockerfile` for a Python Flask app:

```Dockerfile
FROM python:3.10.5-alpine
WORKDIR /app
COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt
COPY . .
EXPOSE 5000
```
Each line creates a **new image layer**.  
Layers allow Docker to cache and reuse previous build steps, making builds faster and more efficient.

***

## Dockerfile: Special Instructions

- **ADD vs COPY:** `COPY` copies local files. `ADD` can extract tars or fetch remote URLs.
- **CMD vs ENTRYPOINT:** `ENTRYPOINT` sets the container’s default executable. `CMD` gives default arguments to the entrypoint.
- **Shell vs Exec form:** Use exec form (`ENTRYPOINT ["prog", "arg"]`) for better signal handling (important for graceful shutdowns).
- **STOPSIGNAL:** Ensures the container receives the right signal (e.g., `SIGINT` for Python programs) when stopping.

***

## Managing Signals: Example

```Dockerfile
FROM python:3.7.13-alpine
COPY main.py main.py
CMD ./main.py
STOPSIGNAL SIGINT
```

Sample `main.py`:
```python
import sys, signal, time

def signal_handler(signum, frame):
    print(f"Gracefully shutting down after receiving signal {signum}")
    sys.exit(0)

signal.signal(signal.SIGTERM, signal_handler)
signal.signal(signal.SIGINT, signal_handler)
while True:
    time.sleep(0.5)
    print("Interrupt me")
```
- `docker stop` sends `SIGTERM`, initiating a graceful shutdown.
- `docker kill` sends `SIGKILL` (forceful termination).

More on [signals](https://docs.docker.com/engine/reference/builder/).

***

## Example: PyTorch Model Training in Docker

**Dockerfile**:
```Dockerfile
FROM python:3.9-slim
WORKDIR /opt/src
COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt
COPY . .
ENTRYPOINT ["python3", "main.py"]
```

**requirements.txt:**
```
torch==1.12.1
torchvision==0.13.1
```

**Run the training:**
```bash
docker build --tag mnist-hogwild .
docker run --rm -it mnist-hogwild
```

Find official PyTorch images with CPU or CUDA support:  
[PyTorch Docker Hub](https://hub.docker.com/r/pytorch/pytorch/tags)

***

## Using GPUs with Docker

- [NVIDIA Container Toolkit Guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)
- Test GPU access:
    ```bash
    sudo docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
    ```

***

## Multi-Stage Builds

- [Python Docker multi-stage article](https://pythonspeed.com/articles/multi-stage-docker-python/)
- [Simple Multi-Stage Guide](https://www.cynnovative.com/simple-multi-stage-docker-builds/)

***



