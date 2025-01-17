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
1.https://stackoverflow.com/questions/46708721/do-all-docker-images-have-minimal-os
2. https://serverfault.com/questions/755607/why-do-we-use-a-os-base-image-with-docker-if-containers-have-no-guest-os
3. https://www.youtube.com/watch?v=1d-LRIZRf5s
4. https://stackoverflow.com/questions/51066146/what-is-the-point-of-workdir-on-dockerfile
5. https://www.youtube.com/watch?v=U1P7bqVM7xM
6. https://stackoverflow.com/questions/21553353/what-is-the-difference-between-cmd-and-entrypoint-in-a-dockerfile
7. https://www.geeksforgeeks.org/how-to-run-a-python-script-using-docker/