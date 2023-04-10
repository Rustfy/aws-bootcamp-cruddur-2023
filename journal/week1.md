# Week 1 — App Containerization

## Required Homework/Tasks

During this week we have dockerized the backend and frontend of Cruddur and test each one individually, then run the hole application (backend+frontend) with docker compose.

### Setup the envirement

- Install the Docker VSCode extention.
- Install the OpenAPI VSCode extention.
- Install the PostgreSQL VSCode extention.

### Containerize the backend (Flask-PY)
The Cruddur backend is writen in flask.
- First we'll create the [`backend-flask/Dockerfile`](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/backend-flask/Dockerfile) which contains the code to build the backend image.
- Build the backend then run it with the required envirement variables set (required in the backend code):
```sh
docker build -t  backend-flask ./backend-flask
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
```
![Run the container](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/01-run-the-container.png)

- We have launched the backend into foreground mode, to run our container in the background we can add the `-d` or `--detach`
```sh
docker run --rm -p 4567:4567 -it -d -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
```
![Run the container in detach mode](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/02-run-the-container-detach-mode.png)

- Unlock the port `4567` then open the URL:
[Backend App](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/03-unlock-the-port.png)

- Test the backend:
![Test the backend](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/04-test-the-backend-container.png)

- Some basic checks:
![Basic checks](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/05-basic-checks.png)

- Stream the logs from the container in real-time:
![Stream the container's logs](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/06-container-logs.png)

- Access to the container shell:
![Container shell](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/07-access-container-shell.png)


### Containerize the frontend (React-JS)
The Cruddur frontend is writen in React-JS, 
- First we'll install the project packages and its dependencies:
```sh
cd ./frontend-react-js
npm i
```

- Create the Docker file of the [`frontend-react-js/Dockerfile`](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/frontend-react-js/Dockerfile)
- Build the container image then run it in a detach mode:
```sh
docker build -t frontend-react-js ./frontend-react-js
docker run -p 3000:3000 -d frontend-react-js
```

### Delete a container/image
By default we can't delete an image or container in use, but if we are sure we can add `--force` arguement to force the deletion.
Below the both way of deletion:
1. First method:
```sh
docker stop <Container-id>
# The below used in case we didn't specify the --rm at the run time, otherwise the container will be automatically deleted
docker rm <Container-id>
```
2. Second method:
This will force the deletion which need to stop the container.
```sh
docker rm <Container-id> --force
```

### Container Orchestration
We have build and run the container of both the backend and frontend apps indeveduelly, but what if we want to run both apps at the time and mange them since one depend on another.
For this we can use a docker orchestration tool like docker compose where we can define the order and how to manage a set of containers under the file [`docker-compose.yml`](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/docker-compose.yml)
![Run Docker compose](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/08-docker-compose.png)
![Cruddur app](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/09-cruddur-app.png)

### Create the notification feature:
![Add Notification feature](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/10-Add-notifications-endpoint.png)

### Adding DynamoDB Local and Postgres:

After inegrating Postgres and DynamoDB into our [Docker Compose file](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/docker-compose.yml) and compose up our app, let 

1. DynamoDB Local:
 We'll test our DynamoDB:
 - Create a DynamoDB table with the below code:
  ```shell
  aws dynamodb create-table \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema AttributeName=Artist,KeyType=HASH AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --table-class STANDARD
  ```
  ![Create DynamoDB Table](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/11-DynamoDB-Create-table.png)

 - Add an item into the created table:
  ```shell
  aws dynamodb put-item \
     --endpoint-url http://localhost:8000 \
     --table-name Music \
     --item \
         '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}}' \
     --return-consumed-capacity TOTAL
  ```
  ![Add an Item into DynamoDB table](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/12-DynamoDB-Create-item.png)
 - List our table:
  ```shell
  aws dynamodb list-tables --endpoint-url http://localhost:8000
  ```
  ![List DynamoDB table](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/13-DynamoDB-list-tables.png)
 - Get Records:
  ```shell
   aws dynamodb scan --table-name Music --query "Items" --endpoint-url http://localhost:8000
  ```
  ![Get DynamoDB Records](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/14-DynamoDB-get-records.png)

2. Postgres:

Install Postgress client into Gitpod
```gitpod
- name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
```
Testing the connection to Postgres:
![Connect to Postgres](https://github.com/Rustfy/aws-bootcamp-cruddur-2023/blob/main/images/week1/15-Postgres-client.png)


## Homework Challenge
