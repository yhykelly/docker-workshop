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

Docker has two ways to bind the data storage into the container:

|   | Named volume  |  bind mount |
|---|---|---|
|What is it | A docker-managed storage| real folder in your host machine|
|Where is it | also somewhere in the host machine, but is managed by Docker | the actual path you declare, managed by yourself|
|Advantage | Docker manages and protects data from being accidentally removed| user has more control; the path is easily shared among coworkers|
| Disadvantage| cannot inspect the data casually; the name volume is local to the host machine, more difficult to be shared; disk usage is opaque|deleting files inside the container deletes them on your host too|

```{bash}
# named volume
# Survives container deletion
-v ny_taxi_postgres_data:/var/lib/postgresql

# bind mount
# location relative to where you run the Docker
-v ./pgdata:/var/lib/postgresql
```

### Useful commands
1. check Docker version
```{bash}
docker --version
```

2. run a docker container, without having to interact with the terminal, e.g.

```{bash}
docker run hello-world

# running background jobs
docker run -d postgres:18

# running batch jobs
docker run --rm my_ingest_job
```

3. run a docker container, having to interact with the terminal (thats why "-it"), e.g.
```{bash}
docker run -it ubuntu bash

# running psql
docker run -it postgres psql -U root

# run Python REPL
docker run -it --rm python:3.9.16 python

# run a shell inside the image
docker run -it --rm python:3.9.16 bash

# override entrypoint (the way the course taught to start python)
docker run -it --rm --entrypoint=bash python:3.9.16
```
`--entrypoint` controls WHAT process starts

4. See what image I have on the machine
```{bash}
docker image ls
```

5. Remove image I have on the machine
```{bash}
docker image rm <IMAGE_ID>
```

6. See stopped containers:
```{bash}
docker ps -a
```
7. Delete all stopped containers:
```{bash}
docker rm $(docker ps -aq)
```
8. `--rm`: indicate cleans up the container after it exits
9. `-d` indicate detached (runs in background)

### Dockerising something - what do we need(#some-markdown-heading)
1. we write in a .Dockerfile
```{Dockerfile}
# base Docker image that we will build on
FROM python:3.13.11-slim

# set up our image by installing prerequisites; pandas in this case
RUN pip install pandas pyarrow

# set up the working directory inside the container
WORKDIR /app
# copy the script to the container. 1st name is source file, 2nd is destination
COPY pipeline.py pipeline.py

# define what to do first when the container runs
# in this example, we will just run the script
ENTRYPOINT ["python", "pipeline.py"]
```

Explanation:

`FROM`: Base image (Python 3.13)
`RUN`: Execute commands during build
`WORKDIR`: Set working directory
`COPY`: Copy files into the image
`ENTRYPOINT`: Default command to run

2. We build up the docker file. Here the image name will be `test`, with a custom "tag" `pandas`. (If the tag isn't specified it will default to `latest`.)

```
docker build -t test:pandas .
```

3. Run the image

```
docker run -it test:pandas <command_line_argument_as_needed>
```

### Example: Running PostgreSQL with Docker
We need:  
1. A running postgres in the background
```
docker run -it --rm \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v ny_taxi_postgres_data:/var/lib/postgresql \
  -p 5432:5432 \
  postgres:18
```

Now, if we want to ingest the data inside this table in this databse running in the container:
```
uv run python ingest_data.py \
  --pg-user=root \
  --pg-pass=root \
  --pg-host=localhost \
  --pg-port=5432 \
  --pg-db=ny_taxi \
  --target-table=yellow_taxi_trips
```

2. **Alternative 1** Use `pgcli`
We use tools as interface connected to the postgres. Here we use `uv run pgcli` (not dockerised here). The port is therefore mapped to the port number at host machine first. The mapped port will be mapped back inside the running container.
```
uv add --dev pgcli
uv run pgcli -h localhost -p 5432 -u root -d ny_taxi
```

**Alternative 2** Use `pgAdmin`. `pgAdmin` can be run in container. Therefore, when we run two containers, we make sure the two containers find each other.
First we creata a Dcoker network
```
docker network create pg-network
```
Then we rerun the postgres in one terminal --
```
# Run PostgreSQL on the network
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v ny_taxi_postgres_data:/var/lib/postgresql \
  -p 5432:5432 \
  --network=pg-network \
  --name pgdatabase \
  postgres:18
```

and run pgAdmin in another
```
# In another terminal, run pgAdmin on the same network
docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -v pgadmin_data:/var/lib/pgadmin \
  -p 8085:80 \
  --network=pg-network \
  --name pgadmin \
  dpage/pgadmin4
```
The container names (`pgdatabase` and `pgadmin`) allow the containers to find each other within the network.
With this network setup, **remember!!** the host is not localhost anymore (we do not connect from host machine now). Host should pgdatabase (the container name)

3. Dockerising the ingestion pipeline as well
Above we learned to run the ingestion from local using uv and map from localhost 5432 to the container port 5432. We can also dockerise the ingestion pipeline and run it as a container. What we have to do is update the Dockerfile. (same logic as [Dockerising something - what do we need](#dockerising-something---what-do-we-needsome-markdown-heading))

```
FROM python:3.13.11-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/

WORKDIR /code
ENV PATH="/code/.venv/bin:$PATH"

COPY pyproject.toml .python-version uv.lock ./
RUN uv sync --locked

COPY ingest_data.py .

ENTRYPOINT ["uv", "run", "python", "ingest_data.py"]
```

Run this in a new terminal, with the other services also running in other terminals. Remember to specify `--network=pg-network` so that it recognise the ingestion share the same network
```
docker run -it \
  --network=pg-network \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pgdatabase \
    --port=5432 \
    --db=ny_taxi \
    --table=yellow_taxi_trips
```

### Docker Compose
Docker Compose is used when multiple services need to run together and we **don't want to run the containers separately in many terminals**. We put all containers and write in a `docker-compose.yaml`, and creates a shared network (e.g. pipeline_default) so services can communicate by name. and we don't need to run everything separately anymore.

```{yaml}
services:
  pgdatabase:
    image: postgres:18
    environment:
      POSTGRES_USER: "root"
      POSTGRES_PASSWORD: "root"
      POSTGRES_DB: "ny_taxi"
    volumes:
      - "ny_taxi_postgres_data:/var/lib/postgresql"
    ports:
      - "5432:5432"

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: "admin@admin.com"
      PGADMIN_DEFAULT_PASSWORD: "root"
    volumes:
      - "pgadmin_data:/var/lib/pgadmin"
    ports:
      - "8085:80"



volumes:
  ny_taxi_postgres_data:
  pgadmin_data:
```
We can now run Docker compose by running the following command from the same directory where docker-compose.yaml is found. Make sure that all previous containers aren't running anymore:

```
docker-compose up
```

After `docker compose up`, we can run the ingestion container taxi_ingest:v001 separately and attach it to the same network (pipeline_default). By using --host=pgdatabase, the ingestion container connects to the Postgres service and writes data into the same persistent database volume.

```
If you want to re-run the dockerized ingest script when you run Postgres and pgAdmin with docker compose, you will have to find the name of the virtual network that Docker compose created for the containers.

# check the network link:
docker network ls

# it's pipeline_default (or similar based on directory name)
# now run the script:
docker run -it --rm\
  --network=pipeline_default \
  taxi_ingest:v001 \
    --pg-user=root \
    --pg-pass=root \
    --pg-host=pgdatabase \
    --pg-port=5432 \
    --pg-db=ny_taxi \
    --target-table=yellow_taxi_trips
```
