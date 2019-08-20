---
title: pgweb client for PostgreSQL databases on Kubernetes
date: 2018-11-09T08:30:48+01:00
draft: false
tags:
  - Go
  - postgres
  - sql
  - Kubernetes
  - Docker
categories:
  - Development
description: >-
  Sometimes it can be helpful to have a graphical interface to manage your database. Whenever I use Postgres, I use pgweb as a client. In one project we ran a Postgres database cluster on Kubernetes and pgweb should be installed as frontend. Unfortunately, the Kubernetes deployment wasn't as easy as I thought.
---

# pgweb client for PostgreSQL databases on Kubernetes

Sometimes it can be helpful to have a graphical interface to manage your database. Whenever I use Postgres, I use pgweb as a client. In one project we ran a Postgres database cluster on Kubernetes and pgweb should be installed as frontend. Unfortunately, the Kubernetes deployment wasn't as easy as I thought.

## About pgweb

Pgweb is a web-based database browser for PostgreSQL, written in Go and works on OSX, Linux and Windows machines. Main idea behind using Go for backend development is to utilize ability of the compiler to produce zero-dependency binaries for multiple platforms. Pgweb was created as an attempt to build very simple and portable application to work with local or remote PostgreSQL databases.

For more information please have a look at the official [project repository](https://github.com/sosedoff/pgweb) on Github.

## Deploy pgweb on Kubernetes

There is already an official docker image for pgweb. Unfortunately the pgweb binary in the container uses a different network interface than needed.  For this reason we need to build our own Docker image for our Kubernetes deployment.

```
FROM alpine:latest as build
ADD https://github.com/sosedoff/pgweb/releases/download/v0.9.12/pgweb_linux_amd64.zip  ./
RUN apk add unzip && unzip pgweb_linux_amd64.zip

FROM alpine:latest
COPY --from=build ./ ./app
EXPOSE 8080
CMD [ "./app/pgweb_linux_amd64", "--sessions" ,"--bind=0.0.0.0", "--listen=8080" ]
```

The arguments can, of course, be adjusted as desired. For Kubernetes it is important that pgweb binds to `0.0.0.0` and not `localhost`. The image can then be pushed into a docker registry.

The Kubernetes deployment file looks like this:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pgweb
  labels:
    service: pgweb
spec:
  selector:
    matchLabels:
      service: pgweb
  template:
    metadata:
      labels:
        service: pgweb
    spec:
      containers:
        - name: pgweb
          image: your/pgwebimage:latest # Change to your image name
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          env:
          - name: DATABASE_URL
            value: postgres:            # Add your postgres connection string
---
apiVersion: v1
kind: Service
metadata:
  name: pgweb-svc
spec:
  type: ClusterIP
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: pgweb

```

Then you can access pgweb either via `kubectl proxy` or `kubectl port-forward`. Of course you can create an ingress object as well to access pgweb from outside the cluster.
