title: Docker tutorial: expose a service
tags: unix, docker

Once you've got docker running, you can start a service and expose it to the outside world.

We'll do this for tomcat7.

First get an container, the basic 120MB ubuntu container for starters

    docker pull ubuntu
    
Then open bin/bash in that container, passing -t allows a psedo tty and -i keeps stdin open even if not attached. You need them to interact with bash.

    docker run -t -i ubuntu /bin/bash
    
You're now in command prompt of your container. Let's use this time to install some stuff.
    
    echo "deb http://dk.archive.ubuntu.com/ubuntu/ precise-security main universe" >> /etc/apt/sources.list
    apt-get install python-software-properties 
    add-apt-repository ppa:webupd8team/java
    apt-get install oracle-java7-installer tomcat7 vim
    vim /etc/default/tomcat7
    # now ensure JAVA_HOME=/usr/lib/jvm/java-7-oracle
    service tomcat7 start
  
Now ensure you get the tomcat7's index.html page by running "wget localhost:8080" in the container. If you can get it, all is well.

You can exit the container by pressing CTRL-D.

If you run "docker ps -a" you'll get all the processes, even stopped ones, which your container now is.

    docker ps -a
    ID                  IMAGE               COMMAND                CREATED             STATUS              PORTS
    9cfedc79aefb        ubuntu:12.04        /bin/bash              About an hour ago   Exit 127                                
    ...
    
You can now commit all the changes to that container. YOu need to do this everytime you want to update your container for later use.

    docker commit 9cfedc79aefb ITS_NAME
    
You should now see this new image when you run docker images

    docker images
    REPOSITORY          TAG                 ID                  CREATED             SIZE
    ITS_NAME            latest              9cfedc79aefb        About an hour ago   550.7 MB (virtual 693.8 MB)

You can start this container up again by running 'docker run -t -i ITS_NAME /bin/bash'.

Notice that only the filesystem changes persist, not the running processes. So your tomcat7 service is no longer running. 

Exit the container again. Now we'll run it again, but this time we'll forward port 8080 on the container, and we'll start tomcat7.

    docker run -i -t -p 8080 ITS_NAME /bin/bash
    service tomcat7 start
    
Now if you open another terminal and run docker ps you can see the forwarding in action.

    docker ps
    ID                  IMAGE               COMMAND             CREATED             STATUS              PORTS
    9b762583b0d0        ITS_NAME:latest     /bin/bash           2 minutes ago       Up 2 minutes        49173->8080 

Note it says we need connect to port 49173 to connect to the 8080 in the container. First we need ip address of docker however.

ifconfig -a should show you something like

    docker0   Link encap:Ethernet  HWaddr blar:blar:blar
              inet addr:172.17.42.1  Bcast:0.0.0.0  Mask:255.255.0.0
              inet6 addr: blar:blar:blar/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:86662 errors:0 dropped:0 overruns:0 frame:0
              TX packets:135316 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:0 
              RX bytes:4610283 (4.3 MiB)  TX bytes:190087628 (181.2 MiB)

So you see you need to look at 172.17.42.1, in this machine's case, to connect to docker. And as such you can access your container's tomcat instance via

    http://172.17.42.1:49173

If you want to be running multiple services and such like, you should look at dockerfiles to automate all this.

