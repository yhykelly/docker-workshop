# Study notes for module 1 - Docker basics
## Docker introduction
### What is Docker?  
- a containerization software that works like a virtual machine, but without its own OS.  
- We run an Docker *image* and turn it into a *container*, which separates its own space and processes from the rest of the OS. The container runs on the host OS, while virtual machine emulate another OS (like a another "real machine") 
- Docker image can be exported to cloud providers as well, such that we can run containers even on cloud. The docker space is accordinly "isolated" up there.
### What is happening behind the scene?
#### I use mac OS and my teammate use Windows. How is it possible for us to collab using Docker?
The container image is actually not mac or Windows based, it is a Linux image. Mac and Windows user run the container with Docker Desktop, under the hood there exists a Linux layer VM

### Why do we use Docker but not only virtual environments?
- At first glance, docker and venv do similar jobs, but they have fundamental differences and serve different purposes:

|   | docker  |  venv |
|---|---|---|
|  What they separate |  the system environment, e.g. Language runtime (can be multiple), System libraries  |  language level libraries, virtual envs stop at the language boundary (i.e. every language gets its own venv) |
|  What is the use | make sure the app is usable across machines (i.e. operating system)  |  make sure the app depend on the particular set of versions of libraries to prevent clash with other projects under the same language|
|  Use case | Team project with mixed OS; Microservices with different languages; Production apps  | you learn a language locally; Data science notebook; experimenting locally;  |
| Management | You write everything in the .Docker file, so the computer does the installations for you when you run the image| you run the installl commands yourself, with activating the venv manually |

#### Beware
After all, the dependency between different system packages or libraries inside the Docker still has to be well thought during development stage. Having Docker does not mean it can guess what is needed.

### What is Volume?
A Docker volume is simply a place to store data that lives outside a container, so the data doesn’t disappear when the container stops or is deleted. Inside Docker, there is actually a folder on the host machine that Docker manages and mounts into the container when the container is run.
