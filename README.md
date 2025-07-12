# Microservices with Kubernetes

## Introduction

A monolith server contains the routing, middlewares, business logic and database access to implement ALL FEATURES of an application.

A single microservice contains the routing, middlewares, business logic and database access to implement ONE FEATURE of an application.

- Biggest benefit: each service is entirely self-contained, meaning if a service crashes for any reasons, the rest of the app is still going to work just fine.
- Biggest drawback: data management, ie. the way storing data inside of a service and how to communicate that data between services.

## Why microservices?

Each service should run independently and not interfere with other services (database per service pattern)

- Database schema/structure might change unexpectedly
- Some services might function more efficiently with different types of database's (SQL or NoSQL)

Comunication strategies between services:

- Synchronous: services communicate with each other using direct requests
- Asynchronous: services communicate with each other using events
  - Event bus: receive events and publish them to listeners (RabbitMQ, Kafka, NATs)
  - pros: having no dependencies on other services & extremely fast
  - cons: data duplication -> extra storage -> cloud storage is incredibly cheap!

## Docker

Docker is a set of platform as a service products that use OS-level virtualization to deliver software in packages called containers.

A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. It should have:

- A base image
- Commands to install additional programs
- A command to run on container startup

## Kubernetes

Kubernetes is a tool for automating deployment, scaling, and management of containerized applications (running a bunch of different containers).

- Kubernetes Cluster is a collection of Nodes (virtual machines). Master is a program to manage everything in the Cluster
- Inside a Node:
  1. A Pod can have one or multiple Containers
  2. A Container is in charge of running a Docker Image
  3. A Deployment monitors a set of Pods, make sure they are running and restarts them if they crash
  4. A Cluster IP Service provides an easy-to-remember URL to access a running Pod
  5. A Load Balancer Service tells the Cluster to reach out to the Cloud Provider to provide a Load Balancer
  6. A Load Balancer serves the outside resquests into An Ingress Controller
  7. An Ingress Controller (ingress-nginx) is a Pod with a set of routing rules to distribute traffic to other Pods through Cluster IP Services inside the Cluster

## Skaffold

Skaffold is a command line tool that saves developers time by automating most of the development workflow in Kubernetes from source to deployment in an extensible way

## App Overview

- Users can list a ticket for an event (concert, sports) for sale
- Other users can purchase this ticket
- Any user can list tickets for sale and purchase tickets
- When a user attempts to purchase a ticket, the ticket is locked for 15 minutes. The user has 15 minutes to enter their payment information
- While locked, no other user can purchase the ticket. After 15 minutes, the ticket should 'unlock'
- Ticket prices can be edited if they are not locked

Resource Types

- User (email: string, password: string)
- Ticket (title: string, price: number, userId: userRef, orderId: orderRef)
- Order (status: enum, expiresAt: date, userId: userRef, ticketId: ticketRef)
- Charge (stripeId: string, amount: number, stripeRefundId: string, status: enum, orderId: orderRef)

Service Types (feature-based design)

- authentication
- tickets
- orders
- expiration
- payments

Events

- User: UserCreated, UserUpdated
- Order: OrderCreated, OrderCancelled, OrderExpired,
- Ticket: TicketCreated, TicketUpdated
- Charge: ChargeCreated

Architecture Design

- A React client app (nextJS)
- Five services (node, express, mongoDB, redis)
- A shared common library (npm module)
- An Event bus (node-nats-streaming)

## Note

- Due to the complexity around user validation, it is necessary to move some of the the middlewares code to a common package that will eventually be pushed into npm `@kei-tickets/common`

- Securely store secrets with Kubernetes: run `kubectl create secret generic jwt-secret --from-literal=JWT_KEY=togetpassthetest`

- Mongodb-memory-server package is a copy of Mongo in memory. This package helps us to test multiple databases at the same time. We can test different services concurringly on the same machine

- Cross namespace communication in Kubernetes:
  - `kubectl get namespace` to get the namespace
  - `kubectl get services -n [nameOfNamespaceAbove]` to tell the service to fetch data from other namespace

## General Information

1. "/etc/hosts": set `127.0.0.1 ticketing.dev`
2. "~/.zshrc": set `alias k="kubectl"`
3. Ingress-nginx is a webserver trying to use secure https connection trying to use 'self sign certificates'. However, Chrome does not trust that kind of certificates so type "thisisunsafe" to Chrome to get over the error
4. Browsers and Postman have different ways to handle cookie & send cookie's data back to the server. Supertest server does not manage cookie automatically & it uses http (not https)
5. Install GCloud SDK to set up kubernetes contexts `gcloud container clusters get-credentials cluster-1`

- step 1: enable google cloud build
- step 2: update the skaffold.yaml file to google cloud build
- step 3: setup ingress nginx on google cloud cluster
- step 4: update hosts file to point to the remote cluster
- step 5: restart skaffold

6. NextJS getInitialProps func can be executed on the client (using Axios) or server (using Kubernetes to reach out to ingress-nginx)
7. Mongoose has built-in Database Transaction for handling transaction issues
