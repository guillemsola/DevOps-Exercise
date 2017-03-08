# Exercise 2

## Using Containers

Containers introduce a lightweight form to define infrastructure and application deployment. In windows we have recently, since Windows 2016, official support for Docker technology. This would be a great oportunity to define and run a multi-container Docker application with Docker Compose. Compose is great for development, testing, and staging environments, as well as CI workflows as it makes really easy to declaratively define a `docker-compose.yml` to create an isolated environment.

We will need a Docker Engine or even a Docker cluster where we can deploy this definition. Jenkins has also nice Docker plugins to communicate with a Docker API to create images and deploy them to an Engine or to the public Docker hub or a private Docker trusted registry.

This will be a proposal for a compose that leverages the official Microsoft images for Docker like [IIS web server](docker pull microsoft/iis) and [SQL Server](https://hub.docker.com/r/microsoft/mssql-server-windows/) for Windows.

We will need a Dockerfile for our web application.

```yml
FROM microsoft/iis

RUN mkdir C:\site

RUN powershell -NoProfile -Command \
    Import-module IISAdministration; \
    New-IISSite -Name "Site" -PhysicalPath C:\site -BindingInformation "*:8000:"

EXPOSE 8000

ADD content/ /site
```

And a compose file to orchestrate the whole deployment including a proxy to balance requests and the networks definition to wire them internally.

```yml
version: '2'
services:
  haproxy:
    image: tutum/haproxy
    depends_on:
      - web
      - sqlserver
    ports:
      – "80:80"
    networks:
      - webnet
  web:
    build: .
    depends_on:
      - sqlserver
    networks:
      - webnet
      - sqlnet
  sqlserver:
    image: microsoft/mssql-server-windows
    networks:
      - sqlnet

networks:
  webnet:
  sqlnet:
```

We can then just launch our system with `docker compose up`. As I'm using Docker compose v2 here we can scale our web server with two instances with `docker-compose scale web=2`.

[Back to home](README.md)