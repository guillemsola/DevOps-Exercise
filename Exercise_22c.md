# Exercise 2

## Upgrade instead of removing

If we want to perform and upgrade we will first need to identify what are the parts of the system that we want to change across deploys. For the SocialGoal application that refers, at least, to the web site that will change every time a developer commits new code that generates a successful build. The rest of the infrastructure and the SQL Server will remain the same. We are assuming that no new SQL scripts are required, otherwise we will need to consider a database migration included with each deployment.

On the previous two exercises there where two different steps, provisioning and deploy. The former was related to infrastructure creation from the virtual machine bootstrapping up to the operating system, the latter about getting the artifacts and deploying the web application. With this we have somehow separated the responsibilities that can be leveraged to upgrade the website version.

We should consider what strategy are we planning to consider with the downtime. If we are fine with the system being down during the deployment process, that won't span more than a few seconds, we could simply remove the website and re-create it again. In fact, that was what the script created in the previous exercises for the Azure deployment were doing.

If we are talking about a production environment we may want to consider other alternatives. We could perform a Blue/Green strategy where a newer version of the application will be deployed in one of the two web servers. We could check that petitions against this server are working as expected before triggering the deployment scripts in the second server.

A more complex scenario will include a proxy in front of the web servers that can be reconfigured on demand. For instance we could use for this an NGinx server or any other proxy able to change its configuration dynamically. With this, for instance, we could deploy the new version in the two servers together with the existing versions but using another port. We could then play with the proxy to redirect some petitions to the new ports. All the petitions will be redirected to the new versions at the time we confirm that no issues are detected.

[Back to home](README.md)