# Challenge Devops
1. Write an appropriate kubernetes object / driver to start a single instance of the PostgreSql database with the following configuration: - User: postgres-Password: AweS0m3 - Contains a database: WORKSHOP
2. Once postgreSql is working, make sure that every time the PostgreSql POD is down, send an email with the notification.
3. Additional Bonus Points, if the PostgreSql POD can create an abse of data from an already configured table.
## - Software Prerequisites

-Install the minikube version V1.0.1
-Install kubectl 1.14.0


## Solving problem 1.

I have decided to use ReplicationController because a postgres db needs to be alive.

Go to the project directory and run the following command to initialize the pod

> kubectl create -f replication.yml


Yaml file content replication.yml
`` `
apiVersion: v1
kind: ReplicationController
metadata:
  name: postgres
spec:
  Replicas: 1
  selector:
    app: postgres
  template:
    metadata:
      name: postgres
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres-ctr
        image: postgres
        lifecycle:
          PreStop:
            httpGet:
              path: "/ sample / mail-failure"
              port: 8080
        ports:
          - containerPort: 5432
            hostPort: 5432
        env:
          - name: POSTGRES_USER
            value: postgres
          - name: POSTGRES_PASSWORD
            value: Aw3s0m3
          - name: POSTGRES_DB
            value: WORKSHOP

`` `

## Solving problem 2

This problem is not clear due to the fact that there is no state down for pods, I suppose that is when a pod is deleted.


I think that there are several ways to solve this problem.

* Using the lifecycle handlers, calling a web api that sends the mail, it would be private (because it is only available in the minikuber vm and we do not need auth) 
`` `
 lifecycle:
          PreStop:
            httpGet:
              path: "/ sample / mail-failure"
              port: 8080
`` `
* Use the exec command: we need to install a mail server inside container
`` `
lifecycle:
      PreStop:
        exec:
          command: ["/ bin / sh", "mail", "mail@sample.com -s pod terminated"]
  
`` `
* Using the wait function of kubectl: if the output "is condition met" run an script based on the operating system in which kubectl is running

> kubectl wait --for-delete pod / postgres

## Solving problem 3

The solution of the problem 3 is to build a custom docker image and publish in a repository, we need to copy our scripts in a folder.

DockerFile
`` `
FROM postgres
# Add the content of the sql-scripts / directory to your image
# All scripts in docker-entrypoint-initdb.d / are automatically
# executed during container startup
COPY ./sql-scripts/ /docker-entrypoint-initdb.d/

`` `


Build the image