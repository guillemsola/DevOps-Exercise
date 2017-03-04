## 1.1.a Repositories and jobs organization 
As release cycles are not tight each component is a product with it's full life cyle and can be modified and updated on demand. For the source code I will create three different git repositories, one for each component, on whatever git server we have like bitbucket, github, TFS... 

Similarly on Jenkins we are going to create three jobs to perform the different activities needed to produce a validated artifact: build, static analysis, tests, delivery... As the three projects share the  same technology the pipelines will be quite similar. This way every time we want to produce a snapshot or release version we will need just to trigger the jenkins job.

If we want to have CI we will need to integrate Jenkins with the SCM of choice with the proper hook plug-in. In this way, and with a proper branch organization it will be possible to cover the whole development life cycle for each product.

## 1.1.b Dependencies management

As one of the three projects that we own is used as a dependency and has an independent life cycle, I will suggest using Nuget, the standard package manager for .NET technologies, to manage them. There are several ways to share Nuget packages, from a shared folder to commercial solution like JFrog Artifactory. Each solution has it's pros and cons concerning costs, simplicity, reliability... Assuming that we don't have another artifact management system like Nexus or Artifactory I will install one of the open source .NET Foundation solutions.

This will be architecture fot the solution

<img src="https://www.draw.io/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Exercise1.1.xml#Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fguillemsola%2FDevOps-Exercise%2Fmaster%2Fmedia%2FExercise1.1.xml" />

We can install a NuGet.Server that is a package provided by the .NET Foundation that creates an ASP.NET application that can host a package feed on any server that runs IIS. In a nutshell it makes a folder on the server available through HTTP(S) (specifically OData). This option it's best for simple scenarios and [is easy to set up](https://docs.microsoft.com/en-us/nuget/hosting-packages/nuget-server). 

I personally would choose the [NuGetGallery](https://github.com/NuGet/NuGetGallery) that is a central dashboard for the packages that we have published within the organization. This tool is [simple to install](https://github.com/NuGet/NuGetGallery#build-and-run-the-gallery-in-arbitrary-number-easy-steps) too and can be used to view package descriptions, versions, maintainers, SMTP alerts... It can be integrated with and LDAP but requires also an SQL server instance.

With a proper Nuget package repository installed the next step is to add a push step on the Jenkins pipeline. For this basically what we need is to add a CMD step in jenkins to pack and push to our Nuget repository.

```powershell
NuGet.exe pack .\src\MySharedLibrary.csproj
NuGet.exe push .\src\bin\Debug\*.nupkg -apikey $api-key -source http://nuget.sga.com:81/nuget/Default
```

Where the API key can be extracted from the NuGetGallery for each user.

Once the package is published we will be able to control from the other projects the required version and restore it at build time.

## 1.2.a

In this scenario we are going to use a jenkins pipeline to provide visualization of the build pipeline and also to provide manual trigger for continuous delivery purposes. In a post build action we will set which project should be build after another. This way is possible to build the different projects all together.

As we want to share the same build number across all the three projects we need a way to share this across all the different jobs. There are several ways to implement this solution. We have to decide whether we want this responsibility to be for jenkins or not, for instance we can set an external API endpoint, file and use these as a seed where we can update or read the build number. This solution will be good if we want to keep incremental build numbers even if we build a single project or all them.

As this is not an explicit requirement and Jenkins offers also tons of different plugins, whe can use the [Parameterized Trigger Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Parameterized+Trigger+Plugin) to solve this problem within Jenkins itself.

With this plugin for example, by defining ```SOURCE_BUILD_NUMBER=${BUILD_NUMBER}``` we will be able to use the variable ```$SOURCE_BUILD_NUMBER``` in subsequent projects to share the same build number.

## 1.2.b

Git has currently two different mechanism to link different projects as a way to manage project dependencies. Git submodules appeared before and are more appropiate for what is refered as a component-based approach, where individual set of files have their own life cycle. Git subtree is newer and more appropiate for a system-based approach, where the whole set has a common life cycle.

Its practical implications are that submodules are not cloned by default and to update them you will need to ```git submodule update```. Also, merging is not really easy with submodules. On the other hand, subtrees get a similar result to submodules, but storing the files in the main repository and merging in changes directly to that repository. That has an important caveat, all of your subproject files are present in the parent repository. Changes made to subprojects have to explicitly be commited its origin if required to be shared across different projects.

It is required to have a good understanding of the projects life cycle to choose the right approach but we can conlcude that, subtree offer more flexibility to modify linked projects letting the responsibility of not mixing parent and sub-project code in commits to developers. Submodules are more appropiate if we don't plan to modify sub-projects code, the downside is that the majority of programming languages nowadays offers package managers for that.

## 1.3

With the above considerations, the decision to go with different release cycles or not shouldn't be a technical but a project life cycle decision. If every time....
