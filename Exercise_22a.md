# Exercise 2

## Base Architecture Definition

SocialGoal is an application build with ASP.NET MVC 5 with SQL Server as backend database. That means that we will need to use windows servers for the ASP.NET technology and the SQL Server. There is an inititiative to [move SQL server to linux](https://www.microsoft.com/en-us/sql-server/sql-server-vnext-including-linux) but is still a work in progress. We have also another requirement where we need two IIS servers.

AS we don't have specific details for the number of concurrent request we are going to receive we are not going to spend too many time in server details. Ideally there should be some kind of load balancing to distribute traffic to the two web servers. As we don't have specific hard disk requirements we are going to assume that the standard server disk is enough just to host the web site. For the database we are going to use another dedicated server for simplicity and, for instance, to set it up in another datalan if aditional security measures are required.

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers lightbox&quot;,&quot;edit&quot;:&quot;https://raw.githubusercontent.com/guillemsola/DevOps-Exercise/master/resources/SocialGoal%20Architecture.xml&quot;,&quot;url&quot;:&quot;https://raw.githubusercontent.com/guillemsola/DevOps-Exercise/master/resources/SocialGoal%20Architecture.xml&quot;}"></div>
<script type="text/javascript" src="https://www.draw.io/embed2.js?s=citrix&fetch=https%3A%2F%2Fraw.githubusercontent.com%2Fguillemsola%2FDevOps-Exercise%2Fmaster%2Fresources%2FSocialGoal%2520Architecture.xml"></script>

This could be a theoretical definition in a DSL for infrastructure as code.

```yaml
server: webnode1
  base_image: windowsServer
  network_segment: prod_db
  allowed_inbound:
    from_segment: internet
    port: 80

server: webnode2
  base_image: windowsServer
  network_segment: prod_db
  allowed_inbound:
    from_segment: internet
    port: 80

server: dbnode
  base_image: windowsServer
  network_segment: prod_db
  allowed_inbound:
    from_segment: prod_web
    port: 1433
  allowed_inbound:
    from_segment: admin
    port: 5985

```


[Back to home](README.md)