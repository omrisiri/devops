### Committing a container

As mentioned before, changes made to a container are not reflected back
on the image it was created from and will, therefore, be lost with the
container itself. It is, however, possible to commit changes made to a
container in the form of a new image. In this exercise, we will make a
small change to a running Alpine container and commit that as our own
image. To get started, start an Alpine container with an interactive
terminal and specify `sh` as the command that has to be executed:

    docker run -it alpine sh
        

This will open a shell inside the container with which we can interact.



Now, we are going to make a (very) minor alteration to the standard
Alpine container by leaving our mark.

    echo "I was here" > proof.txt
        

To check if the above command executed successfully, type the following:

    cat proof.txt
        

This should print the `I was here` message. Now exit the container by
typing `exit`.



As explained before, if we were to start up a new container using the
Alpine image, the proof.txt file wouldn’t be there. To create an image
that does contain our file, we need to create an image using the
`commit` command. To do that, first look up the ID of the container
using `docker ps -a` (when we exited the container, it was stopped, so
we need the `-a` argument here). To create the new image, use the
following command:

    docker commit <container> my-custom-alpine
        

Upon entering this command, Docker will create an image from the
specified container and print its new unique ID as output. Verify its
existence by using the command to list all images. If everything went
correctly, it should now also list a `my-custom-alpine` image. Because
we did not specify a tag it was created with the `latest` tag as a
default.



Now let’s run this freshly created image. Start a container from the new
image, and output the contents of the file. Or if you want, you could
start the container using an interactive shell to view that the file is
there.

    docker run my-custom-alpine cat proof.txt
        

If this prints out `I was here` it means you have successfully created
your own image! Please note that this is a really terrible way to create
an image. Using a Dockerfile is much more efficient, and you can easily
keep track of your changes. Creating an image using the `commit` command
is mostly just useful when debugging or prototyping.

### Creating a simple image using a Dockerfile

While the above method works, it can be quite time-consuming to manually
modify containers and save them as images. And with a Dockerfile you can
create much smaller images if you do it correctly. Therefore, the
preferred way or creating new Docker image is through the use of a
Dockerfile. This file describe all steps that are necessary to create a
specific image.



Now let’s create the same image we made with `commit` using a
Dockerfile. On your machine, create a file with the name `Dockerfile`, with no file
extension. Make sure it is in an empty directory. In the same directory,
create the proof.txt with the text `I was here`. Now open the Dockerfile
in your favorite editor (unless it is emacs ;) ), and write the
following:

    FROM alpine
    LABEL maintainer="replace this with your name or email"
    COPY ./proof.txt /proof.txt
        

We used a couple of instructions here to describe a new Docker image.
The first instruction, `FROM`, is used to declare the image we want to
use as our base. You always need a base image, as minimal as it may be.
In this case, we are building on top of the `latest` version of the
`alpine` image (remember that Docker uses `latest` automatically if not
specified). The second instruction, `LABEL`, is an optional instruction
which can add metadata to your image. In this case we only added one
key-value pair, but you can add as many as you like. 
`Note: Docker also has the MAINTAINER instruction, but it is deprecated`
The last instruction, `COPY`, simply copies the file (COPY src dest). You could
also use `ADD` to copy the file, but add can do much more. If you `ADD`
an archive, it automatically unpacks it, while `COPY` doesn’t. It also
allows you to use a url as the source.



To actually build the image, open your terminal, go to the directory
containing your Dockerfile and run the following command:

    docker build -t my-first-dockerfile .
        

You should see an output like this:

    Sending build context to Docker daemon  3.072kB
    Step 1/3 : FROM alpine
    ---> 7328f6f8b418
    Step 2/3 : LABEL maintainer "your name or email"
    ---> Running in 6f8d453b244e
    ---> ffe36d15abb0
    Removing intermediate container 6f8d453b244e
    Step 3/3 : COPY ./proof.txt /proof.txt
    ---> 579e521c3f10
    Removing intermediate container 3d0de735f6ff
    Successfully built 579e521c3f10
    Successfully tagged my-first-dockerfile:latest
        

The first thing we see is ’Sending build context to Docker daemon
3.072kB’. What Docker does is copying the entire directory of the
Dockerfile to the Docker daemon. Which is only 3.072kB now, but if you
have a huge directory it can take a long while. If you do have a huge
directory, but don’t need everything for your build, you can create a `.dockerignore`
file, which works similarly to a .gitignore file. Then you see three
steps that are executed, with their output (if they have any). Each step
has its own unique hash, which refers to a cached layer (a set of
filesystem changes) of a docker image. So if you run the exact same
instruction again, it will use the cached layer instead of executing it
again. Run the build command again and you will see this:

    Sending build context to Docker daemon  3.072kB
    Step 1/3 : FROM alpine
    ---> 7328f6f8b418
    Step 2/3 : LABEL maintainer "your name or email"
    ---> Using cache
    ---> ffe36d15abb0
    Step 3/3 : COPY ./proof.txt /proof.txt
    ---> Using cache
    ---> 579e521c3f10
    Successfully built 579e521c3f10
    Successfully tagged my-first-dockerfile:latest
        

As you can see, it now uses the cache for both steps.



Now let’s test our custom image:

    docker run my-first-dockerfile cat proof.txt
        

It should output the contents of the file. Remember that you can view
all your local images by running the `images` command.

### Creating an Angular Docker image

Now we will do a bit more than just copy a file. We already created an
app with the Angular CLI for you, using the command `ng new app`. Now we need to
create a Dockerfile that will first install all dependencies, then run
the Angular application. Create the Dockerfile in the angular-app
directory, with the following contents:

    FROM     node:alpine
    WORKDIR  /opt/app
    COPY     app/ .
    RUN      npm install
    EXPOSE   4200
    CMD      ["npm","start"]
        

The first instruction should be familiar now, as we create an image
using `node:alpine` as a base. After that, we specify the working
directory inside the to-be-created image with `WORKDIR`. This is the
directory that will be used as the starting point for subsequent
commands such as `COPY`, `RUN` or `CMD`. The `COPY` copies the contents
of the angular-app to the current directory in the image, which is
/opt/app as we specified it as our `WORKDIR`. Furthermore, `RUN`
specifies a command that is to be executed when *building the image*.
Important to note is that the commands specified with `RUN` are always
run as the root user by default. There is a `USER` instruction available
as well which lets you switch the user with which commands are run. In
this particular example, we want NodeJS to install all the dependencies
for the application, but as this happens during the building of the
image it won’t have to be repeated every time a container is started.
That means that all the dependencies are available and ready when a
container is started. The `EXPOSE` instruction informs Docker that the
container listens on the specified network port at runtime. The app will
run on port 4200, which is why we expose it. You still need to provide
the `-p` option if you want to access it. Finally, we specify the
default command for the container in case none is given in the
`docker run` command. This command is also executed in the working
directory, which is /opt/app in our case.



Next, build the image (this may take a while), and run it without giving
it a command. Remember to use the option -it if you want to be able to
use `Ctrl+C` to stop the container. If you don’t, remember to stop the
container later using `docker stop <container-name>`.

    docker run -p 4200:4200 <image-name>
        

Let’s head over to <http://localhost:4200/> to test it, and you should
see the Angular app.



Instead of using the `CMD` instruction, we can also use `ENTRYPOINT`.

    ENTRYPOINT  ["npm","start"]
        

This allows you to send extra arguments to the entrypoint instruction by
passing it to your `docker run` command:

        docker run -it -p 4200:4200 <image-name> my extra args
        

This will now run `npm start my extra args`. Obviously this does nothing, but you can see that
the extra commands are passed. This does introduce a problem though,
when you want to start an interactive shell.

    docker run -it <image-name> sh
        

This doesn’t work now, because `sh` is passed through to the npm
command. You can however open an interactive shell to a running
container:

    docker exec -it <container id or container name> sh
        

### Reducing the image size

When creating Docker images, it is important to try to reduce the image
size. A real good example of why you would want to have small images is
when using containers in your Continuous Integration/Deployment
pipeline, e.g. GitLab. GitLab can use Docker images to build an
application, so it downloads a Docker image for every step in your
pipeline. It’s really easy to create large images without even noticing.
This is because of the layers Docker uses. The less instructions you
have in your Dockerfile, the faster your build, and it will most likely
also be smaller in size. It is also important to remove any unused
files, as these will also be saved in the layer.



If we look at the Angular image we created before, it’s almost 267MB. To
reduce its size, we can clear the npm cache after running npm install:

    FROM     node:alpine
    WORKDIR  /opt/app
    COPY     app/ .
    RUN      npm install && \
             npm cache clean --force
    EXPOSE   4200
    CMD      ["npm","start"]
        

If we build the image now, you can see its size has been reduced to
232MB. In this case it isn’t much, but that is also because we have a
small application. In some cases it can save a lot of space. Also notice
how the two npm commands are combined in one `RUN` instruction. This
reduces the amount of layers that have to be built in the construction
of the image, as each instruction in a Dockerfile requires a layer.



Another example is when using apt-get. For the same reasons as stated
above, it is recommended to run every apt-get install command in one
`RUN` instruction as opposed to adding a `RUN` instruction for every
package you want to install. And of course clean the apt-get list:

    RUN apt-get update && apt-get install -y \
        aufs-tools \
        automake \
        build-essential \
        curl \
        dpkg-sig \
        libcap-dev \
        libsqlite3-dev \
        mercurial \
        reprepro \
        ruby1.9.1 \
        ruby1.9.1-dev \
        s3cmd=1.1.* \
        && rm -rf /var/lib/apt/lists/*
        



The same applies to using instructions like the `COPY` instruction.

    COPY file1 /somedir
    COPY file2 /somedir
        

versus

    COPY file1 file2 /somedir
        

Even though the resulting image will be the same, this way Docker will
require an extra step, and therefore an extra layer, when building. This
can slow down the building process.



This doesn’t mean that you should put everything inside a single
instruction, but you should be aware of what happens when you add an
extra one. Try to keep your Dockerfile maintainable as well, so find a
balance between maintainability and image size.



See
<https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/>
for a list of best practices when creating a Dockerfile.



The instructions we used so far in our Dockerfiles are only a small set
of options you have. You can find more at
<https://docs.docker.com/engine/reference/builder/>.

# Volumes

Docker containers are generally meant to be easily shut down and
replaced. Therefore, they should not contain any data that has to be
persistent. It is generally a bad idea to have data stored inside your
image, because (obviously) the image size will increase. Luckily, there
is a way that allows containers to persist data outside themselves, so
you won’t lose it when shutting down a container. This is done through
the use of *volumes*.

### Attaching a volume to a single container

A volume can simply be a directory on your host system that you mount in
a Docker container. This can be done by using the `-v` argument when
starting a container. Create a new directory and name it `data_dir`. Additionally,
create one or more files in this new folder, for example a text file.
Then, navigate to the parent of this directory in the terminal and type
in the following command (continue reading for the Windows versions of this command):

    docker run -it -v $(pwd)/data_dir:/opt/host_data alpine sh
        

With this command, we bind the `data_dir` directory to the directory
`/opt/host_data` inside the container. The `$(pwd)` command is not
Docker-specific but a terminal command used to retrieve the current
directory we’re in, which has to be the one which has the data\_dir
directory as its subfolder. This is needed because the `-v` option
requires absolute paths. Please note that the `$(pwd)` command will not
work in Windows. In Windows you need to use a path with backslashes
(forward slashes if you use git-bash):

    docker run -it -v C:\Users\<user>\Documents\data_dir:/opt/host_data alpine sh

    IN POWERSHELL:
    docker run -it -v ${PWD}\data_dir:/opt/host_data alpine sh

    IN COMMAND PROMPT:
    docker run -it -v %cd%\data_dir:/opt/host_data alpine sh
    

If you go to that folder in your container and type `ls` you should see
a list of the files you created in the data\_dir folder earlier. The
reverse also works: adding or modifying files inside this directory in
the container will reflect directly on the files of the host system.
Feel free to try it with the following command:

    echo filecontent > filecreatedincontainer.txt
        

You should now see the new file in the directory on your host machine.
You could use this directory to save data you’d want to keep after
stopping the container, or that you want to share between containers.

### Using containers as volumes

When sharing data between containers with volumes, it is generally
better to have a dedicated *Data Volume Container* for that purpose. You
can use any image you like for that. In the next exercise, we are going
to create two containers that will share a directory from a third one.
First, we create the Data Volume Container based on an Alpine image:

    docker create -v /mydata --name datastore alpine
        

The `create` command creates a container without starting it.
Furthermore, we name the container ourselves so that we can conveniently
refer to it later on.



For the next step, let’s start the containers that will use our created
data volume. In two separate terminals, execute the following command:

    docker run -it --volumes-from datastore alpine sh
        

If you use `ls` in these containers, you’ll see the directory `mydata`
amongst them. Anything in this directory will be shared between the
containers that mounted the datastore container as a volume.



To test this, run the following command in one of the containers:

    echo This is from container 1! > /mydata/message.txt
        

Then, in the other container, use `cat /mydata/message.txt` to print the
contents of that file. Stopping these containers and running new ones
with the data\_store volume attached should all result in the same:
everything in mydata is persisted in the datastore container. You can
also use the `--volumes-from` command multiple times to mount volumes
from different containers into one.



In the first exercise we specified the mounting point on our host
directory ourselves by using the syntax `/host_dir:/data_dir`. In the
second exercise, we only wanted a directory called `/mydata` in the
container. Docker created a directory somewhere on your system which it
subsequently bound to the container. This is called an *anonymous
volume*. This directory is not automatically removed when the container
for which it was originally created is. You can instruct Docker to do
this by appending the `-v` argument when removing a container, for
example:

    docker rm -v datastore
        

If you don’t do that, you may end up with *“dangling” volumes*, or
volumes that are no longer referenced by any container. You can also
remove these *“dangling” volumes* using the command:

    docker volume prune
        



You can list all volumes with the `docker volume ls` command. Docker
uses unique hashes as names for anonymous volumes, much like the ID’s
given to images and containers. You can list all dangling volumes by
adding some extra arguments to this command, like this:

    docker volume ls -f dangling=true
        

After that, you can use `docker volume rm <volume-name>` to permanently
delete any volume you want.

# Analysing containers

Docker provides a number of commands to analyse the various aspects of
your containers. Run a few containers to try these commands out. If you
would like to know how much resources each of your running containers is
using, you can see this using the `stats` command:

    docker stats
        

which will show a live stream of the CPU and memory usage of each of
your running containers (you can stop this with `Ctrl+C`).



To view the logging of a specific container, use the following command:

    docker logs <container>
        

Here, you can specify the container using either the full *container
name* as it appears under the `NAMES` column in the `docker ps` command,
or the `CONTAINER ID`. For the latter option, you don’t have to provide
the entire ID, just enough so that it is not ambiguous in your current
situation. For example, if we had two containers running with ID’s
`0xab123...` and `0xac97j...` respectively, the following command would
succeed:

    docker logs 0xab
        

while this would fail:

    docker logs 0xa
        

This command is especially useful if you want to see the logging of
detached containers.



One last command that is worth mentioning is `inspect`, which can be
used as follows:

    docker inspect <container>
        

in which the container can be specified in the same way as with the
`logs` command. This prints quite a bit of low-level information
regarding the container, such as its status, configuration and network
settings. You could, for example, omit the forwarding of ports using
`-p` and simply approach the server using its own IP address found in
the `IPAddress` property.

# Docker 102 Workshop

We also have a Docker 102 Workshop, which will be on the 26th of July.
It will show you how to use Docker multi stage builds, and Docker
Compose. So the final exercise in this workshop, is to sign up for it.
Go to <https://www.jcore.com/event/docker-102-workshop/>.

# Further reading

In this workshop, we covered the very basics of Docker. While this
should be enough to get started, there still is a lot more available.
Docker itself offers various tools, such as Docker Compose and Docker
Swarm, that can help you to easily start a set of containers or manage
clusters. Furthermore, Docker is often well integrated in third party
software such as Kubernetes, GitLab, and OpenShift. Documentation for
the standard Docker engine as well as other tools provided by Docker
itself can be found at <https://docs.docker.com/>.

