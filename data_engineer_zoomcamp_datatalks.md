# Data Engineering Zoomcamp (by DataTalks Club)
### Duration : 1.5 hours

## Day 1
* To run an Ubuntu container in docker : `docker run -it ubuntu bash`.The docker run command automatically pulls and runs the image without the need to run docker pull first. 

```
# to run python 3.9 in containerized environment
docker run -it python:3.9

# when we want to run pip install, we cannot do it directly in the python shell but have to do it in the bash terminal, hence we specify entrypoint as bash
docker run -it --entrypoint=bash python:3.9

```

* Dockerfile : A text file which contains sequence of steps to build a docker image. You can imagine as if this is a terminal where you are typing all your commands, but instead of typing it one at a time, you are typing it all at once and then running it all using `docker build` followed by `docker run` . Below is example of a simple dockerfile

```
### Sample DOCKERFILE
### The folder we are running from just has 2 files excluding DOCKERFILE - hello.py (a simple flask app) and requirements.txt
FROM debian:buster

COPY . /myproject

RUN pip install flask # RUN pip install -r /myproject/requirements.txt

CMD ["python","myproject/app.py"]

```
* Once we create DOCKERFILE, we build it to create docker image, and then run the image to have the container as shown below

```
# build the dockerfile using docker build .
# by using tag, it is easier to refer to image, compared to using the image id
docker build --tag hello-flask:1.0 .

# see list of images available
docker images

# run the image with port binding
docker run -p 5000:5000 <imageid/tag>

```

* Base image : Most Docker images aren’t built from scratch. Instead, you take an existing image, called base image, and use it as the basis for your image using the FROM command in your Dockerfile

* In regular command line, if you cd somewhere, it stays there until you change it. However, in a Dockerfile, *each RUN command starts back at the root directory*. That's a gotcha for docker newbies, and something to be aware of. So not only does WORKDIR make a more obvious visual cue to someone reading your code, but it also keeps the working directory for more than just the one RUN command(similar to cd in normal terminal).

```
### DOCKERFILE with WORKDIR (compare with previous dockerfile)

FROM debian:buster

WORKDIR /myproject

COPY . .

RUN pip install flask # RUN pip install -r requirements.txt

CMD ["python","app.py"]

```

* You should use WORKDIR instead of proliferating instructions like RUN cd … && do-something (RUN cd /myproject && pip install -r requirements.txt), which are hard to read, troubleshoot, and maintain.

* Layer caching : Common layers are not pulled multiple times. For example, if we run docker pull python:3.9 and then docker pull python:3.10, then the layers in 3.10 image common to 3.9 image are not pulled but used directly from 3.9 image. To see how to leverage caching to build more efficient DOCKERFILE see reference 3

* ENTRYPOINT : To use when you want to append some additional command to command entered in docker run.
Assume following ENTRYPOINT in DOCKERFILE
`ENTRYPOINT [ "npm", "init" ]`
Now, If I run docker run -t node install
It will append the `npm init` to `install` and overall command run is `npm init install`



### Doubts
1. What is the difference bw docker-compose run and docker-compose up?
2. To build a Python application is docker, what is difference b/w FROM ubuntu and FROM python:3.9
3. Do all docker images need an OS? Are all Docker images based on linux? Is the python:3.9 image based on linux?
4. If containers do not have a guest OS, then why do we base the container on an OS image such as `FROM ubuntu`?
5. Why use WORKDIR in the DOCKERFILE?
6. In dockerfile with workdir, shouldn't workdir command come after copy?
7. Is the default shell for python:3.9 image a python shell instead of bash terminal?

### References
1. https://stackoverflow.com/questions/46708721/do-all-docker-images-have-minimal-os
2. https://serverfault.com/questions/755607/why-do-we-use-a-os-base-image-with-docker-if-containers-have-no-guest-os
3. https://www.youtube.com/watch?v=1d-LRIZRf5s
4. https://stackoverflow.com/questions/51066146/what-is-the-point-of-workdir-on-dockerfile
5. https://www.youtube.com/watch?v=U1P7bqVM7xM
6. https://stackoverflow.com/questions/21553353/what-is-the-difference-between-cmd-and-entrypoint-in-a-dockerfile
7. https://www.geeksforgeeks.org/how-to-run-a-python-script-using-docker/



## Day 2
### Duration : 2.25 hours

### Learnings
* Environment variable : a user-definable value that can affect the way running processes will behave on a computer. In docker environment variables will be redeclared in each new containers

* ENV (keyword in dockerfile): Set environment variables with some default value.

* ENV values from the Dockerfile will be overridden via the CLI (using -e flag during docker run or env files)

* ARG (keyword in dockerfile): Create build-time variables

* ENV vs ARG :  ARG is for building your Docker image i.e. a variable defined using ARG can be assigned only during `docker build` and not during `docker run`. ENV is for future running containers. ENV is mainly meant to provide default values for your future environment variables, and these environment variables can be assigned new values during `docker run`

* Docker Volume : Volumes are persistent data stores for containers, created and managed by Docker. In simple words, when you stop and start a container, the data inserted previously is not lost (hence used to solve issue of data persistence)
```
# volume-name is path of data in host system
# mount-path is path of data in docker container
docker run --volume <volume-name>:<mount-path>
```

* The command to create a postgres container with volume is as follows (will be used to store nyc taxi data)

```
# the same thing can also be done using docker-compose.yml
docker run -it \
    -e POSTGRES_USER="root" \
    -e POSTGRES_PASSWORD="root" \
    -e POSTGRES_DB="ny_taxi" \
    -v C:\Users\Dell\Documents\ny_taxi_postgres_data:/var/lib/postgresql/data
    -p 5432:5432
    postgres:13

```

* Errors faced
    * ``docker: invalid reference format`  Cause: `-v ($pwd)/ny_taxi_postgres_data:var/lib/postgresql/data` and `-e POSTGRES_USER = "root"` Solution : remove space around = 
    * `invalid mount config for type "volume": invalid mount path: 'var/lib/postgresql/data' mount path must be absolute`.
    * `docker: Error response from daemon: create $(pwd)/ny_taxi_postgres_data: "$(pwd)/ny_taxi_postgres_data" includes invalid characters for a local volume name, only "[a-zA-Z0-9][a-zA-Z0-9_.-]" are allowed. If you intended to pass a host directory, use absolute path`. Solution : Give full path instead of $(pwd) in Windows,
    * `ModuleNotFoundError: No module named 'psycopg2' while running upload_data.py ` Solution: pip install psycopg2


```
pip install pgcli

pgcli -h localhost -p 5432 -u root -d ny_taxi

# test connection to running database
SELECT 1; 
```

* Following is the Python code to insert tripdata from csv file into Postgres database

```
### upload_data.py 
import pandas as pd

from sqlalchemy import create_engine
from time import time

df = pd.read_csv('yellow_tripdata_2021-01.csv', nrows=100)

print(pd.io.sql.get_schema(df, name='yellow_taxi_data', con=engine))

# postgresql://user:password@hostname/database_name
engine = create_engine('postgresql://root:root@localhost:5432/ny_taxi')

df_iter = pd.read_csv('yellow_tripdata_2021-01.csv', iterator=True, chunksize=100000)

while True: 
    t_start = time()

    df = next(df_iter)

    df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
    df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)
    
    df.to_sql(name='yellow_taxi_data', con=engine, if_exists='append')

    t_end = time()

    print('inserted another chunk, took %.3f second' % (t_end - t_start))

# Using pgcli we see total of 1369765 records using select count(*) from yellow_taxi_data;

```

### Doubts
1. Where exactly in the DOCKERFILE are the environment variables used when we type `docker run -e POSTGRES_USER=root`
2. What is difference bw ENV and ARG in dockerfile? 


### References
1. https://vsupalov.com/docker-arg-env-variable-guide/
2. https://docs.docker.com/reference/dockerfile/
3. https://stackoverflow.com/questions/33935807/how-to-define-a-variable-in-a-dockerfile
4. https://stackoverflow.com/questions/41916386/arg-or-env-which-one-to-use-in-this-case
5. https://stackoverflow.com/questions/34809646/what-is-the-purpose-of-volume-in-dockerfile
6. https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
7. https://github.com/DataTalksClub/nyc-tlc-data
8. https://stackoverflow.com/questions/45682010/docker-invalid-reference-format
9. https://stackoverflow.com/questions/9353822/connecting-postgresql-with-sqlalchemy

## Day 3

### Duration : 3 hours
### Learnings
* Pgcli : Pgcli is a command line interface for Postgres, built using Python

* Port : A virtual point where network connections start and end. 

* Port number : A 16-bit integer that serves as a unique identifier for a specific proces/service/application on a networked device.

* Port mapping : By default, when we create or run a container using docker create or docker run, containers on bridge networks don't expose any ports to the outside world. The -p flag makes a port available to services outside the bridge network.
```
# -p HOST_PORT:CONTAINER_PORT
-p 8080:80	Map port 8080 on the Docker host to TCP port 80 in the container.
```

* Inspect Postgres database using pgcli as shown below


```
# Connect to running database by providing : Host, port, username, database
pgcli -h localhost -p 5432 -u root -d ny_taxi

select * from information_schema.tables;


select column_name, data_type from information_schema.columns where table_name='yellow_taxi_data'


```

* Pgadmin : A tool for managing Postgres. Provides graphical interface to create and update database objects (tables, schema etc)

* PgAdmin 4 docker container has exposed port 80 and 443 by default (refer DOCKERFILE of pgadmin). Hence we map port 8080 of our computer to port 80 of pgadmin

```

docker run -it \
    -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
    -e PGADMIN_DEFAULT_PASSWORD="root" \
    -p 8080:80 \
    dpage/pgadmin4

```


* Docker Network : A virtual network created by Docker to enable communication between Docker containers. If two processes are running on same machine/container, then no need of network, but two different containers are like 2 separate machines, hence we need to create a network. If two containers are running on the same host they can communicate with each other without the need for ports to be exposed to the host machine.

* TO connect pgadmin to postgres container, we need to create a docker network and connect both the containers to the same docker network
as shown below (environment variables have been e)
```
# create a network
docker network create pg-network

# connect an existing container to the network
docker network connect pg-network competent_germain

# docker rename competent_germain pg_database_1

# see details about network like driver, list of connected containers, etc
docker inspect pg-network

# connect a new container with network using network flag
docker run -it \
    -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
    -e PGADMIN_DEFAULT_PASSWORD="root" \
    -p 8080:80 \
    --network pg-network \
    --name pg-admin \
    dpage/pgadmin4

```

* Steps for using pgadmin are
    1. Login using email and password listed above
    2. Click on Add New Server
    3. Give a name to connection in General tab
    4. In Connection tab, enter following details
        * Host name: IP address of postgres container
        * Port : 5432
        * Username : username of postgres database
        * Password : password of postgres database
    5. Click on Save
    6. Select Query Tool icon at left top 


* `Error : Unable to connect to server, name does not resolve in pgadmin4` Solution: Make sure the postgres container is running. Get the id of the container and do `docker inspect <container-id>`. Copy the IPv4 address and paste it into the Host Name field

* Data serialization : Process of converting an object into a stream of bytes to more easily save or transmit it. 

* YAML (YAML Ain't Markup Language) : A human-readable and human-writable data interchange format for storing and transmitting the information

* YAML has two top-level elements, an object and array
    * Object : a collection of key-value pairs. Each key is followed by :
    * Array : ordered list of values, with each item preceded by -
 JSON and YAML are very similar

* Service : abstract definition of a resource within an application for example a database or a web app frontend or a web app backend

* A service can be run by one or multiple containers. With docker you can handle containers and with docker-compose you can handle services.

* Using docker-compose, we can create the same containers above instead of having to create each container separately, as shown below. Docker compose saves us from writing kilometre long run statement from the terminal, and can be considered as a wrapper around docker cli which makes our lives easier

```
# docker-compose.yml
services:
    pgdatabase:
        image: postgres:13
        environment:
            - POSTGRES_USER=root
            - POSTGRES_PASSWORD=root
            - POSTGRES_DB=ny_taxi
        volumes:
            - "./ny_taxi_postgres_data:/var/lib/postgresql/data:rw"
        ports:
            - "5432:5432"
    
    pgadmin:
        image: dpage/pgadmin4
        environment:
            - PGADMIN_DEFAULT_EMAIL=
            - PGADMIN_DEFAULT_PASSWORD=root
        ports:
            - "8080:80"
        
# To run : docker-compose up -d
# To down: docker-compose down
```
* If we do not use network top level element, docker compose automatically creates a network

* By default docker just manages the volume to host mapping for you i.e. where in the host system the volume is stored.

* (Is this correct?)We add a volume for pgadmin so that we don't have to continually add the connection every time you bring it up. The pgadmin_conn_data volume is separate from your PostgreSQL volume (ny_taxi_postgres_data). The PostgreSQL volume is used to persist data from your PostgreSQL database, while the pgadmin_conn_data volume is used to persist data from your PgAdmin application. 

* Using network top level element in dcoker-compose we can name the network name in the compose file (useful when we want to access it from another container)
```
docker volume ls

docker volume inspect volume-name

docker network ls

```

* 3 major docker concepts : container, network, volume

### Doubts
1. What is difference bw process, service and application?
2. What is differnece bw physical and virtual network?
3. What is the use case for docker network drivers other than bridge (like host, none, etc)?
4. How to add a docker container which has already been created to a new docker network?
5. Why ports in docker-compose wrapped in double quotes? In general when to and not to use double quotes in docker-compose yml file?
6. What is difference bw service and container in docker-compose?
7. For volumes why no need of full path in docker compose for pgdatabase?
8. What is the difference bw volume and bind mount?

### References
1. https://docs.docker.com/engine/network/
2. https://superuser.com/questions/209654/whats-the-difference-between-an-application-a-process-and-a-service
3. https://github.com/pgadmin-org/pgadmin4/blob/master/Dockerfile
4. https://stackoverflow.com/questions/48046585/docker-pgadmin-4
5. https://www.geeksforgeeks.org/basics-of-docker-networking/
6. https://stackoverflow.com/questions/50721424/how-to-add-containers-to-same-network-in-docker
7. https://stackoverflow.com/questions/57109494/unable-to-connect-to-server-pgadmin-4
8. https://deepsource.com/glossary/yaml
9. https://stackoverflow.com/questions/35565770/difference-between-service-and-container-in-docker-compose
10. https://www.reddit.com/r/docker/comments/11fm9zr/help_me_understand_dockercompose_named_volumes/
11. https://kinsta.com/blog/docker-compose-volumes/


## Day 4
### Duration : 1 hour

* Cloud Infrastructure : Hardware/software in cloud such as storage, server, compute, networking etc

* Infrastructure as Code(IaC) : Practice where infrastructure is managed and provisioned using code rather than manual processes. By storing infrastructure code in version control system, we can ensure infrastructure changes are trackable

* Terraform : IaC tool which allows users to provision and manage infrastructure resources across various cloud platforms (and even on prem)

* HCL (HashiCorp Configuration Language) : Language used in Terraform

* State file : Stores info about infrastructure's configuration and status (i.e. things like resources that have been created, their current properties) Named `terraform.tfstate `

* Important Terraform CLI commands:
    * `terraform fmt`: Format terraform code
    * `terraform init` : Beginning of project or we decide to add more providers or change the version of existing ones 
    * `terraform plan` : Compares the code with the state file to identify and highlight resources that will be created, updated, or deleted if we choose to execute the current version of the code. ALso goes through configuration files and identify syntax errors/version mismatch
    * `terraform apply` : Actually execute code to create/delete resources on cloud
    * `terraform destroy`

* Basic workflow : init -> plan -> apply

* Plugin :  A piece of software that extends functionality of existing software/provides additional functionality eg. Grammarly, OneTab, Sceenshot of entire page(Chrome), Adblocker

* Provider : Plugin that enables interaction with an API and thus allows it to manage infra on any platform eg. AWS Provider to manage resources on AWS

* Resource :  Infrastructure objects (like virtual networks or compute instances)

* Module : Container for multiple resources that are used together (can include resources from the same provider or different providers)

* Below is code to create a simple GCP Cloud Storage Bucket using Terraform
```
### main.tf

terraform {
    required_providers{
        google = {
            source="hashicorp/google"
            version="5.5.5"
        }
    }
}

provider "google" {
    credentials = "./keys/my-creds.json"
    project = "gcp-project-id"
    region = "us-central1"
}

resource "google_storage_bucket" "demo-bucket" {
    name = "bucket-name" # must be globally unique across all of gcp
    location = "US"
    force_destroy = true

}

```


### Doubts
1. What does declarative syntax mean? And how is it different from imperative?
2. How exactly is terraform cloud agnostic and be used to automate multi cloud deployment? How does it map say cloud storage in GCP to S3 in AWS? 

### References
1. https://spacelift.io/blog/terraform-tutorial
2. https://dzone.com/articles/an-introduction-to-terraforms-core-concepts



## Day 4 and 5
### Duration : 1 + 1 hour
* Apache Airflow and Kestra are an open-source data orchestration and scheduling platforms to manage complex data flows

* Orchestration (remember orchestra where coordination is imp.) : Coordination and management of multiple applications and services, stringing together multiple tasks in order to execute a larger workflow or process

* Data orchestration : Programmatically author, schedule, and monitor workflows.

* Task : Step/action to be performed eg. download data, transform data. Two types
    * Flowable tasks: Tasks which orchestrate the flow eg. io.kestra.plugin.core.flow.Parallel
    * Runnable tasks: Tasks which perform actual work/take some action eg. io.kestra.plugin.scripts.shell.Commands to execute shell commands


* Some properties of task include: id, type,  

* Namespace : Logical grouping of flows eg. based on team

* Flow : Container/Grouping for set of tasks, their inputs and outputs and associated error handling. Defines the order in which tasks are executed and how they are executed (i.e. parellel, sequential)

* Variable : Key value pair to reuse value across tasks

* Namespace variable : Variables whose scope is limited to specific namespace. Namespace variables have to be defined via Kestra UI and not via YML file

* Trigger : Mechanism to automate execution of a flow (schedule driven or event driven)

* States: Control status of your workflow execution

* Template engine : A template engine enables you to use static template files in your application. At runtime, the template engine replaces variables in a template file with actual values

* Pebble : Java templating engine, similar to Jinja in Python

* Expression : Used to dynamically pass data to the workflow in real time. Uses Pebble, hence expression must be wrapped in {{}}. 

* Some variables which exist by default and need not be created explicitly are `flow`, `inputs`, `outputs`, `tasks`. In example below the inputs variable is used

```
## inputs.name is an expression which allows Kestra to use the name passed by end user in real time in the logging task
id: myflow
namespace: company.team

inputs:
    - id: name
      type: STRING

tasks:
    - id: hello
      type: io.kestra.plugin.core.log.log
      message: "Hello {{ inputs.name }}"


```

* If a variable uses a Pebble expression, we must use render() function when using that variable

```
id: myflow
workspace: company.team

variables:
    file: "{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv"


tasks:
    - id: set_label
      type: io.kestra.plugin.core.execution.Labels
      labels:
        file: "{{render(vars.file)}}" # since file uses Pebble expression, use render function
        taxi: "{{inputs.taxi}}"

```

* There are multiple types of tasks available, and each has its own set of  properties. Refer Kestra documentation

```
id: postgres-taxi
namespace: zoomcamp

inputs:
    - id: taxi
      displayName: Select taxi type
      values: ['yellow','green']
      defaults: 'yellow'

    - id: year
      displayName: Select year
      values: ['2019','2020']
      defaults: '2019'

    - id: month
      displayName: Select month
      values: ['01','02', '03', '04', '05', '06', '07']
      defaults: '01'


variables:
    file: "{{inputs.taxi}}_tripdata_{{inputs.year}}-{{inputs.month}}.csv"
    
```

* No need of remembering all the task types, just know the high level logic to implement, rest can be gooten from documentation

* In staging table, we create a unique row id for each row by concatenating all the column values and using md5(instead of using uuid)


### Doubts
1. What is purpose of staging table?
2. What are static files (in context of Jinja)?
3. What is difference between static and dynamic task?
4. What is truncate table in SQL? What is update statement in SQL?
5. What is the difference bw merge and insert in SQL?

### References
1. https://medium.com/geekculture/airflow-vs-prefect-vs-kestra-which-is-best-for-building-advanced-data-pipelines-40cfbddf9697
2. https://expressjs.com/en/guide/using-template-engines.html
3. https://kestra.io/docs/expressions

