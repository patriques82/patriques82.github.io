---
layout:     post
title:      Dockerizing Payara server with PostrgeSQL
date:       2020-07-30 11:57:00
author:     Patrik Nygren
summary:    How to dockerize a Payara full server with PostgreSQL
categories: jakartaee, payara, docker
thumbnail:  cogs
tags:
 - jakartaee
 - payara
 - docker
---

This is a follow up to the "KotlinEE with graphql on Payara" post. We will containerize the application we setup before. To do it we have to extend the payara/server-full image, since we need it to have a postgresql jdbc driver inside and a database connection-pool to connect to the postrgesql container.

We first download the jdbc driver from [jdbc.postgresql.org](https://jdbc.postgresql.org/download.html), then we build the image.

```dockerfile
FROM payara/server-full:5.201

# copy driver to enable postgres connection
COPY postgresql-42.2.14.jar $PAYARA_DIR/glassfish/lib

# create connection pool for postgres called postgres-pool
RUN echo 'create-jdbc-connection-pool --datasourceclassname org.postgresql.ds.PGConnectionPoolDataSource --restype javax.sql.ConnectionPoolDataSource --property user=postgres:password=secret:DatabaseName=app:ServerName=postgres:port=5432 postgres-pool' \
 > $POSTBOOT_COMMANDS
```

```bash
docker build --tag=payara-with-postgres .
```

Now that we have the payara-with-postgres image we can setup the whole application with docker-compose. To use our war file we link the `build/libs/graphqldemo.war` to the payara/deployments. We also of course need to setup the same user, password,database name and servername as we did when creating the connection pool.

```docker
version: "3"
services:
  payara:
    image: payara-with-postgres
    ports:
      - "8080:8080" # Web app
      - "8181:8181" # HTTPS listener
      - "4848:4848" # HTTPS admin listener
      - "9009:9009" # Debug port
    restart: always
    volumes:
      - ./build/libs:/opt/payara/deployments # link war to container deployments folder
    networks:
      - app-network
    depends_on:
      - postgres

  postgres: # this must match the servername setup when creating the connection pool
    image: postgres
    volumes:
      - app-data:/var/lib/postgresql/data/ # persist data even if container shuts down
    networks:
      - app-network
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=app

networks:
  app-network:
volumes:
  app-data:
```

For convinience we also sat up a custom network and volume to isolate the app logic. To see it work in action log in to the payara server on [https//localhost:4848]([https//localhost:4848]). Find the jdbc connection under Resources/JDBC/JDBC Connection Pools and then click on the postgres-pool, and try to ping it.