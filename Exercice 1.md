## 1.1 Repositories and jobs organization 
As release cycles are not tight each component is a product with it's full lyfecyle and can be modified and updated on demand. For the source code I will create three different git repositories, one for each component, on whatever git server we have like bitbucket, github, TFS... 

Similarly on jenkins we are going to create three jobs to perform the different activities needed to produce a validated artifact: build, static analysis, tests, delivery... As the three projects share the  same technology the pipelines will be quite similar. This way every time we want to produce a snapshot or release version we will need just to trigger the jenkins job.

If we want to have CI we will need to integrate jenkins with the SCM of choice with the proper hook plugin. In this way, and with a proper branch organization it will be possible to cover the whole development lifecycle for each product.

## 1.1.b Dependencies management

As one of the three projects it's a dependency with its own cycle but that we own, I will suggest using Nuget, the standard package mananger for .NET technologies. There are several ways to share nuget packages, from a shared folder to comercial solution like JFrog Artifactory. Each solution has it's pros and cons concerning costs, simplicity, reliability... Assuming that we don't have another artifacts management system like Nexus or Artifactory I will install a Nuget.Server.

NuGet.Server is a package provided by the .NET Foundation that creates an ASP.NET application that can host a package feed on any server that runs IIS. In a nutshell it  makes a folder on the server available through HTTP(S) (specifically OData). This option it's best for simple scenarios and [is easy to set up](https://docs.microsoft.com/en-us/nuget/hosting-packages/nuget-server).

We have also the option to install the [NuGetGallery](https://github.com/NuGet/NuGetGallery) to have a nice central point to manage the internal packages we have published. This tool is [simple to install too](https://github.com/NuGet/NuGetGallery#build-and-run-the-gallery-in-arbitrary-number-easy-steps)  and can be used as a repository for descriptions, track versions, maintainers...



The decision to go with different release cycles 
