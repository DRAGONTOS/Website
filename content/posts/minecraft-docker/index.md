---
title: "Minecraft server in a docker container!"
ate: 2024-04-02
draft: true
description: ""
tags: ["debian", "server", "vm", "docker"]
---
Setting up a Minecraft server with a Docker container, would it be with a proxy or with a regular one.

# Minecraft server in a docker container!

## Prerequisites 
you will need to have an debian server installed with docker on how to do that you can follow my debian 12 server [post](https://kaleyfischer.xyz/posts/debian-server-install/).

## Setting it up 
To set it up you need to clone my [repo]():
```
cd ~/docker && git clone https://git.kaleyfischer.xyz/DRAGONTOS/minecraft-docker
```

## Server Setups 
There is a lot of choice for setting up a server you could use a proxy or a regular install and
both have a lot of option cus you could go with forge, fabric, quilt, papermc and even purpurmc and for proxies
there is bungeecord and velocity but for this [post]() i will go with papermc and velocity on version 1.20.1 because fabic/quilt can have
problems with mods not working perfectly or being extremely buggy (if you do want a guide for them then you can dm on [twitter]()).

### Regular Server setup 
Setting up a normal server setup isn't so hard and i will even use a quiple plugins like essentials and worldedit.

#### Docker Configuration
You first ofc need to configure docker to use [PaperMC]() and i will guide you line by line on how to do that.
First you need to open it you do that by:
```bash
nano docker-compose.yml
```

This section is normally used for setting up a proxy cus you use docker network routing but you can ignore this for this setup (do not remove this ofc).
```bash
networks:
  customnetwork:
    external: true
    name: minecraft-stack
```
For the CHANGETHIS sections you just need to enter the name of your server so that if you run multiple servers with
docker that it doesn't collide name wise.
```bash
  minecraft-(CHANGETHIS):
    restart: always
    image: minecraft-(CHANGETHIS)
    build: ./minecraft-docker
```
The -Xmx2048M is for how much ram you allocate to the server you could change this to 4gb if 2gb isn't enough and server.jar is the 
server you can also change this if needed but i recommend renaming papermc server jar to server.jar
```bash
    command: ["java", "-Xmx2048M", "-jar", "server.jar", "true"]
    networks:
      customnetwork:
        ipv4_address: 192.168.16.(CHANGETHIS)
```
This where you need to put the data ./data and 8001 is remote console because you can't traditionally
access the minecraft console becuase it's running in docker and for setting up rcon i will walk you through that in the next [step]() also what is important to know is if you have multiple server running with rcon 
then you ofc need to change the port for the outside so change the CHANGETHIS section in "CHANGETHIS:8001".
```bash
    volumes:
      - "./data:/root/minecraft"
    ports:
      - "25565:25565"
      - "CHANGETHIS:8001" # RCON (REMOTE CONSOLE)
```

#### Setup 
Now you need to setup the actual server with [PaperMC]() and you need to configure rcon as mentioned [earlier]().


for setting up papermc you need to get the papermc software from [here]() then you need to put it in ./data rename it to server.jar or change the name in docker-compose.yml then you need to test run it to see if it worked
```bash
docker compose up # ctrl + c to exit
```
it should give you an error becuase you didn't accept the [eula]() yet. (the eula is located in ./data/eula.txt) now that the server is installed and working you should test it by runnin the container as a daemon and connecting to the ip to run it you can execute this command:
```bash
docker compose up -d
```
now that the server is running you should connect to it with minecraft on version 1.20.1 and you should know how to get the ip with following my post on setting up a debian server with docker [post]()
when connect you might be convused on how to get op to get that you should start an rcon session to go over that you can follow this [post]() about it but you do first need to enable it in system.properties
and to do that you need to open it with your favorite text editor. 
```bash
nano ./data/server.properties
```
then you need to do afew thing like setting a password, the port and enable it ofc 
- **PORT:** It should be on line 4 and i recommend to set it to 8001.
- **ENABLING:** Line 34 and you should set it to true
- **PASS:** Line 42.

#### Plugins 
You can now install plugins and the plugins i will go over in this post is essentialsx and worldedit to install them you just need to put them in .data/plugins/
and then run this command to restart the server:
```bash
docker compose down && docker compose up -d
```
<table>
    <thead>
        <tr>
            <th>Title</th>
            <th>Description</th>
            <th>Link</th>
        </tr>
    </thead>
    <tbody>
         <tr>
            <td>EssentialsX</td>
            <td>EssentialsX is the essential plugin suite for Minecraft servers, with over 130 commands for servers of all size and scale.</td>
            <td><a target="_blank" href="https://essentialsx.net/downloads.html">site</a></td>
        </tr>
         <tr>
            <td>EssentialsX Chat</td>
            <td>Chat formatting, local chat.</td>
            <td><a target="_blank" href="https://essentialsx.net/downloads.html">site</a></td>
        </tr>
         <tr>
            <td>EssentialsX Spawn</td>
            <td>Spawnpoint control, per-player spawns.</td>
            <td><a target="_blank" href="https://essentialsx.net/downloads.html">site</a></td>
        </tr>
        <tr>
            <td>FastAsyncWorldEdit</td>
            <td>Blazingly fast world manipulation for artists, builders and everyone else.</td>
            <td><a target="_blank" href="https://www.spigotmc.org/resources/fastasyncworldedit.13932/">site</a></td>
        </tr>
    </tbody>
</table>

#### Conclusion
You should now have setup a [PaperMC]() server with EssentialsX, Worldedit and have rcon working and running.  

## Wrapping It Up 
I hope that you now have a working Debian 12 server installed with Docker and a running Nginx site!
And for help, you could always DM me on Twitter for the time being until I have my own Masadon account.

