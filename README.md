# Introduction

This exercise can be seen with [Github pages here](https://guillemsola.github.io/DevOps-Exercise/)

# Exercise 1
We have two web projects that both use a common library that we also own. All projects are .net and have their own repository in git. We want to put a Jenkins system in place.

1. If we want to have a separate release cycle for all three components…
  - a. How would you organize the repositories and the build jobs?
  - b. Would you use any type of package system? How?
2. If we want to use the same release cycle for all three projects…
  - a. How would you specify the same build number for all components using pipeline jobs?
  - b. What are the pros and cons of linking projects using subtrees or submodules in this context?
3. What would be your recommendation? Same release cycle for all components or not?

**[Solution for the first exercise](Exercise_1.md)**

# Exercise 2
We have this web application https://github.com/asg123/SocialGoal
1. Set up a Jenkins build **[Solution](Exercise_21.md)**
2. Whenever we have a successful build, we want to deploy it automatically, following an Infrastructure as code approach.
  - a. Describe a base architecture with one database server and two IIS servers, in a infrastructureascode definition
  - b. Using the description in previous point, set up an automatic deployment...
    1. In our own datacenter (e.g. vCenter, or any other tool of your choice) **[Solution](Exercise_22bi.md)**
    2. In a cloud provider (Amazon, Azure) **[Solution](Exercise_22bii.md)**

> NOTE: In both scenarios, you can take advantage of containers technology (e.g. Docker, Kubernetes...)

  - c. How would you change the approach if with each deployment we want to upgrade the same running environment instead of removing the existing and creating a new one?

3. After the deployment, we want to run a selenium test suite that lives in the same repository of the application. How would you implement this?

# IMPORTANT!
For each exercise, please
  - Document any assumption you make because of lack of information in the questions
  - Specify all tools used, to provide an end to end complete answer
  - Provide detailed steps on how you would implement a working solution, like a howto guide. This includes any configuration settings, scripts, etc. Feel free to add screenshots or any other reference you find informative
