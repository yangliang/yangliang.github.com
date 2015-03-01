---
layout: post
title: Open Source Continuous Deployment
date: '2014-12-22T19:33:00.003-08:00'
author: Kevin L
tags:
modified_time: '2014-12-23T00:22:16.793-08:00'
thumbnail: /images/post_opensourceCI/thumb_2014-12-22-192016_979x846_scrot.png
---

>"Continuous Deployment is simple: just ship your code to customers as often as possible" -Timothy Fitz

#### Going from code to deployment in the easiest and most pain-free way is the goal. 

This mostly includes:

 - Issue-Tracking System
 - Version Control
 - Continuous Integration
 - Build and Unit Tests
 - Deployment to Production
 - Fail-Fast


Services have been written to get the job done, but most are paid and hosted solutions. 
Github Enterprise, Jira, Gitlab, Travis-CI,  etc...

All great tools for their use, but sometimes it' best to host the services yourself for no fees. The tools used here are all free and open source.

---

#### Step 1. Version Control & Issue Tracking

Gogs is a Github clone written in Go. It has evolved into a respectable application. It features organizations, teams, issue tracking, repository hooks, and way more. Check it out. The project is currently hosted on Github.

![Gogs](/images/post_opensourceCI/2014-12-22-192016_979x846_scrot.png)


Installation is easy,

 - .1. Bring up a Linux Distro of your choice.

 - .2. Install git (obviously)
    [wget the latest Binary](http://gogs.io/docs/installation/install_from_binary.html)

 - .3. 
  ``` 
   $ mv output.zip /srv/ \
   && unzip /srv/output.zip
  ```
 - .4. 
  ```
   $ bash /srv/gogs/start.sh
  ```
 - .5. Navigate to http://[IP-address]:3000

And that's it. Setup a reverse proxy, ssl, and run it in a screen/tmux/nohup for better performance.

---

#### Step 2. Continuous Integration


Drone is a Continuous Integration service that utilizes Docker to build and test applications. Pull requests get tested before merging as well. Build status can be emailed, announced in IRC, or sent in other addon chat protocols

It can OAuth logins with Github, Github Enterprise, Gitlab, Bitbucket, Gogs, and other services. So there is no need to register on the server, you authorize Drone access to your account and it will query for all your repositories.

![Drone](/images/post_opensourceCI/2014-12-22-202249_1190x902_scrot.png)

 Installation takes < 5 minutes.

 - .1. Bring up a Debian-based Linux Distro of your choice.

 - .2. 

    ```
   $ wget http://downloads.drone.io/master/drone.deb
   $ sudo dpkg -i drone.deb
    ```

 - .3. Add your gogs url to /etc/drone/drone.toml

    ```
    [gogs] 
    url="http://[your-gogs-url]/"
    ```

 - .4. Login to Drone

 - .5. Have a drink

 ![Drone-Build](/images/post_opensourceCI/2014-12-22-202622_420x977_scrot.png)

To enable a repository for CI on Drone, check enable in the repository list, then commit a .drone.yml file to your repository.

---
#### Step 3. Status & Deployment

Drone-Wall 			 
Wall displays are an easy way to see the project status. (Drone-Wall)[https://github.com/drone/drone-wall] is perfect for a TV on the wall.

Installation can be done using [Docker](https://www.docker.com/)

![Drone-wall](/images/post_opensourceCI/2014-12-22-202249_1190x902_scrot.png)

 - .1. Get your Drone API Key under "Account Settings" in Drone

 - .2. Fill in the variables here

    ```
    $ docker pull scottwferg/drone-wall
    $ docker run -d -p 3000:3000 -e API_SCHEME=HTTP -e API_DOMAIN=[DRONE_DOMAIN] \
    -e API_TOKEN=[API_KEY] -e API_PORT=80 scottwferg/drone-wall
    ```

 - .3. Have another drink.

![Drone-pic](/images/post_opensourceCI/Snapchat--5268095761595617025.jpg)

---
Now how do we get this successfully built code to production?

The .drone.yml file has the ability to trigger a deployment at the successful completion of your build.

Add the Drone public ssh key provided in your Account settings to your production server.
Add deploy parameters in the .drone.yml file to ssh in, clone the master branch, and restart services.
More guidance can be found on the [Official documentation](https://github.com/drone/drone/blob/v0.2.1/README.md#builds) and some [basic scripting](http://docs.drone.io/ssh.html)

---
#### Conclusion
Great!
Now we have an open source, free, and automated environment that will version control, issue track, build, test, and deploy code. This is perfect continuous deployment. Code that breaks will not be deployed, so chances of failure go down greatly. And since we are continuously deploying, no reason to wait a week to push fixes if we do fail.

The code to deployment pipeline should be very fast. In the event of a failure, fixes need to get deployed as quickly as possible. "Set a fixed goal for the total commit-build-deploy process. At IMVU that goal was roughly 10 minutes, at Canvas that goal was 5 minutes." - Timothy Fitz
Drone's build & testing using Docker is extremely quick for my python-flask application.



See how Etsy goes from code to deployment with [Deployinator](https://codeascraft.com/2010/05/20/quantum-of-deployment/)

Inspired by:
[http://timothyfitz.com/2009/02/10/continuous-deployment-at-imvu-doing-the-impossible-fifty-times-a-day/](http://timothyfitz.com/2009/02/10/continuous-deployment-at-imvu-doing-the-impossible-fifty-times-a-day/)
[http://timothyfitz.com/2012/11/25/paths-to-continuous-deployment/](http://timothyfitz.com/2012/11/25/paths-to-continuous-deployment/)


