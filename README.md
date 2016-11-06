# Socker = Sitecore :heart: Docker

Is is now possible to run Sitecore completely in Docker natively, you don't have to mess around with databases, IIS or anything, you don't even have to have SQL Server or IIS installed on your machine.
 
This repository shows how a solution running Sitecore is wired up for development with the following features:

- Databases is persisted between restarts
- Project output are automatically synced into running containers when changes are detected
- Load balancing multiple Sitecore instances
- Streaming log output

So why develop Sitecore solutions with Docker?

- Getting complete solutions up and running is *very* fast
- Isolate dependecies and versions (like Mongo, Solr, Windows patches) used for each solution without polluting your machine
- Minimize disk space usage, you only have 1 copy of each Sitecore version as a Docker images
- Potentially run your code inside the *same* container locally as in production 

## Prerequisites

1. Windows 10 Anniversary update with latest updates
2. Docker for Windows v1.12.3-beta29.3 or later (beta channel: [https://download.docker.com/win/beta/InstallDocker.msi](https://download.docker.com/win/beta/InstallDocker.msi))

## Preparations

>NOTE: Base images are build locally in this example, but in a real life senario you would also push to an remote private repository like 
hub.docker.com, quay.io or an internal one so images can be shared within your organization.
Unfortunately it has to be **private** repositories due to Sitecore licensing terms so we can't share images in the community.

#### Using Sitecore v8.1 rev. 160519

1. Extract "Data" and "Website" from "Sitecore 8.1 rev. 160519.zip" and place them in `/docker/sitecore-81rev160519/Sitecore`
2. Build private images:
	
````
docker build -t sitecore-iis .\docker\sitecore-iis
docker build -t sitecore:8.1.160519 .\docker\sitecore-81rev160519
````

#### [OPTIONAL] Using another Sitecore version

1. Extract "Data" and "Website" from "SSitecore 8.2 rev. 160729.zip" and place them in `/docker/sitecore-82rev160729/Sitecore`
2. Build private images:
	
	````
	docker build -t sitecore:8.2.160729 .\docker\sitecore-82rev160729
	````

3. Change the version number in the FROM statement in "/docker/web/Dockerfile"
4. Replace "/src/WebApp/Web.config" with the Web.config from "SSitecore 8.2 rev. 160729.zip"

#### [OPTIONAL] Using Traefik load balancer

If you want to try out an loadbalanced environment you should also build:

````
docker build -t traefik:win .\docker\traefik-win
````

Then you can use `docker-compose --file .\docker-compose.scale.yml up` to start your containers.

### Solution

1. Place "license.xml" in `/docker/web/Sitecore/Data`
2. Place Sitecore databases from "Sitecore 8.1 rev. 160519.zip" in `/data`

## Daily usage

- Start containers:

````
docker-compose build
docker-compose up
````

- Open IP of web container in browser (IP is in the compose output or use `docker inspect`)
- Open Socker.sln.
	- Add files, edit code, build - watcher script updates the running containers...
	- Refresh browser, repeat...
- When you're done, press CTRL+C to stop

If you do not care about the output from containers you can start in "detached" mode with `docker-compose up -d` and then use `docker-compose stop` or `docker-compose down` to remove everything. 

>TIP: You can attach to the web containers to watch output from Sitecore logs with `docker exec socker_web_1 powershell C:/Sitecore/Scripts/Stream-Log.ps1`