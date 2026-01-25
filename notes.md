# Study notes for module 1 - Docker basics
## Docker introduction
### What is Docker?  
- A containerization platform that provides isolated environments similar to virtual machines, but without a separate OS kernel.  
- We run a Docker *image* and turn it into a *container*, which separates its own space and processes from the rest of the OS.
- A container runs on the host OS kernel, while a virtual machine emulates an entire computer with its own OS.
- Docker image can be exported to cloud providers as well, such that we can run containers even on cloud. The docker space is accordinly "isolated" up there.
### What is happening behind the scene?
#### I use mac OS and my teammate use Windows. How is it possible for us to collab using Docker?
The container image is actually not mac or Windows based, it is a Linux image. On macOS and Windows, Docker Desktop runs containers inside a lightweight Linux virtual machine under the hood.

### Why do we use Docker but not only virtual environments?
- At first glance, docker and venv do similar jobs, but they have fundamental differences and serve different purposes:

|   | docker  |  venv |
|---|---|---|
|  What they isolate |  OS-level environment: language runtimes, system libraries, tools, filesystem  |  Language-level libraries only (per language) |
|  What is the use | make sure the app behaves the same and reproducible across machines and operating systems |  ensure the app depends on a specific set of library versions, avoiding conflicts between projects using the same language|
|  Use case | Team project with mixed OS; Microservices with different languages; Production apps  | you learn a language locally; Data science notebook; experimenting locally;  |
| Management | You write installation steps in the .Dockerfile, and Docker automatically installs dependencies when building the image | You manually create and activate the virtual environment and run install commands yourself. |

#### Beware
After all, the dependency between different system packages or libraries inside the Docker still has to be well thought during development stage. Having Docker does not mean it can guess what is needed.

### What is Volume?
A Docker volume is simply a place to store data that lives outside a container, so the data doesn’t disappear when the container stops or is deleted. Inside Docker, there is actually a folder on the host machine that Docker manages and mounts into the container when the container is run.
