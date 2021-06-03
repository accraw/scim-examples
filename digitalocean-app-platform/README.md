# Deploying the 1Password SCIM bridge using DigitalOcean App Platform
This document will describe how to deploy the 1Password SCIM bridge using DigitalOcean's App Platform.

## Deployment Overview
App Platform is DigitalOcean's new, fully managed solution for deploying applications via a code repository. The idea behind App platform is that there is less of
a focus on the manual management of containers in a given environment, and more of a focus on deploying applications for immediate access. After testing
the deployment of the SCIM bridge with App Platform, it is now being added as a deployment option for those using DigitalOcean to host the SCIM bridge.

Deploying the SCIM bridge with App Platform comes with a few benefits:
* App Platform will provide and host the URL for your SCIM bridge; you will not need to setup an A record or prepare a name for a URL as noted in [PREPARATION.md](https://github.com/1Password/scim-examples/blob/master/PREPARATION.md)
* App Platfom will host the SCIM application for a low cost of $5/mo. An additional $5/mo or $6/mo will be utilized for the Droplet created for the Redis container.
* There's no need to manage the container that the SCIM bridge will be running on.


## Preparation and Deployment
To get started with deploying the SCIM bridge using App Platform, you'll need:

* Access to your organization's DigitalOcean tenant.
* Access to your organization's Github account in order to fork this repository.
* Access to create a Droplet for Redis in your organization's DigitalOcean tenant.


### Step One: Setting up Redis

Before you deploy the SCIM bridge application using App Platform, a redis database must be created first, so that you can add the connection details for your database to the application at setup. There are two options for setting up a redis database: creating a Droplet in DigitalOcean and installing redis onto it or using DigitalOcean's Managed Redis database solution.

To create a Droplet:

* Under Manage in the left-hand navigation menu, select Droplet or select the Create dropdown menu in the top right corner of your DigitalOcean tenant and select Droplet.
* Choose an image for your container.
* Choose a plan for your Droplet. (The Basic (shared CPU) tier is sufficient, but choose what's best for your organization)
* Choose a datacenter region.
* Under Finalize and Create, you will only need 1 Droplet.
* Once you've configured the other settings on this page to your liking, click Create Droplet.

Once the creation process of your Droplet is complete:

* Click on the hostname of your new container from your list of Droplets.
* Click on Console. (ensure that the credentials for your image are set and that you can log into the container)
* At this point, you will want to install redis on your Droplet. DigitalOcean provides detailed documentaion on how to install redis on each of its provided images. Documentation can be found [here](https://www.digitalocean.com/community/tutorial_collections/how-to-install-and-secure-redis)
* For the ```Binding to Localhost``` step in the redis documentation, you will want to ensure that you allow all connections initially, so that the SCIM application can make a connection to your Droplet. After the successful deployment of your SCIM application, you can lock down access to your redis Droplet, ensuring that your SCIM application only has access to that Droplet.

If you prefer to use DigitalOcean's Managed Redis Database solution:

* Under Manage in the left-hand navigation menu, select ```Databases``` or select the Create dropdown menu in the top right corner of your DigitalOcean tenant and select Databases.
* Choose Redis as your Database Engine.
* Under Choose your Configuration, leaving the ```Machine Type``` set to the ```Basic Nodes``` option is sufficient.
* Choose a Datacenter.
* Once you've configured the other settings on this page to your liking, click ```Create a Database Cluster```.

Once the creation process of your managed database is complete:

* Click on the hostname of your new container from your list of managed databases.
* In the top right corner, click on the ```Actions``` dropdown menu and select ```Connection details```.
* Under the ```Public Network``` settings, you will need to take note of the hostname as well as the provided port number.
* You can secure your database's inbound connections using DigitalOcean's ```Getting Started``` tutorial or by selecting ```Secure this database cluster by restricting access``` under the ```Trusted Sources``` section on the Overview page. You will want to complete this step after you've successfully deployed the application in Step Two, so that you can add the ip address of the application's container to that section. 


### Step Two: Building and Deploying using App Platform

Now that a redis Droplet has been created, you can start the deployment process of the SCIM application.

* Under Manage in the left-hand navigation menu, select Apps or select the Create dropdown menu in the top right corner of your DigitalOcean tenant and select Apps.
* Select Launch Your App on the splash page. If you've already started using Apps, select Create App in the top right corner of the page.
* Choose Github as your source. (You may be prompted to walk through an authorization process for your Github account and your DigitalOcean tenant)
* Choose the repository that contains the files for the DigitalOcean App Platform deployment.
* Choose the corresponding branch.
* You can choose to allow or deny Autodeploy code changes.
* Click ```Next```.
* To configure your app, you will want to set two environment variables: ```OP_REDIS_URL``` and ```OP_SESSION```. 
 * If you are using a Droplet, ```OP_REDIS_URL``` should contain the following: redis://[ip or hostname of redis Droplet]:6379 
 * If you are using DigitalOcean's managed database solution, ```OP_REDIS_URL``` should contain the following: redis://[ip or hostname of redis Droplet]:[provided port number]
 * ```OP_SESSION``` should contain the base64 encoded version of your scimsession file. Run the following command to generate the scimsession: cat /path/to/scimsession| base64 | tr -d "\n"
 * You've successfully run the command when the base64 encoded version of your scimsession is returned in the terminal. Copy and paste the contents and paste them as the value of the OP_SESSION variable. (Do not copy the % sign at the end of the contents)
* Set the HTTP port for the app to 3002.
* Click ```Next```.
* Name your application.
* Select a region for the application/container.
* Click ```Next```.
* The Basic tier of the App Platform is suffient enough for the SCIM bridge.
* Under Containers, the ```Basic Size``` is defaulted to the ```1 GB RAM | 1 vCPU``` option, however the ```512 MB RAM | 1 vCPU``` option is sufficient for this deployment.
* ```Number of Containers``` should be set to 1.
* Select ```Launch Basic App```.
* The App will begin the build and deploy process. Once complete, you should receive an alert that states ```Deployed Successfully``` and the URL for the SCIM bridge will be made available on the Apps Dashboard. (You may need to refresh your page if the URL is not yet visible at this point)
* Click the URL link and enter the bearer token for your SCIM bridge to start Provisioning tasks.
* Ensure that you add the provided URL and the bearer token to your IdP of choice.


You can also deploy your application using the Deploy to DigitalOcean button below, which is a quick link that will start the process of Step Two:

[![Deploy to DO](https://www.deploytodo.com/do-btn-blue.svg)