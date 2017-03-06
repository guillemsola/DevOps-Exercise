#Exercise 2

## Upgrade instead of removing

If we want to perform and upgrade we will first need to define what are the parts of the system that change across deploys. For the SocialGoal application that refers to the web site code that will change every time a developer commits new code. The rest of the infrastructure and the SQL Server will remain the same. We are assuming that no new SQL scripts are required, otherwise we will need to consider a database migration included with each deployment.

On the previous two exercices there where two different steps, provisioning and deploy. The former was about creating the infrastructure, from the virtual machine bootstrapping including the operating system. The latter was about copying the artifacts and deploying the web application. With this we have somehow separated the responsibilities that can be used to upgrade the website verison.

We should consider what strategy are we planning to consider with the downtime. If we are fine with the system being down during the deployment process that will be about some seconds we could simplify remove the website and create it again. This was what the script created in the previous exercise for the Azure deployment was doing.

If we are talking about a producion environment we may want to consider other alternatives. We could perform a Blue/Green strategy where a newer version of the application will be deployed to one of the two web servers. We could check that petitions against this server are working as expected to triger the deployment script on the second server.

A more complex scenario will include a proxy before the front-end systems that can be reconfigured. For instance we could use for this an NGinx server or any other proxy able to change its configuration dynamically. With this for instance we could deploy the new version in the two servers together with the existing versions using another port. We could then play with the proxy to redirect some petitions to the new ports. All the petitions will be redirected to the new versions at the time we confirm that no issues are detected.   

[Back to home](README.md)