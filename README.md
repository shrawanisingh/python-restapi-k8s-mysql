# Developing a Flask API with GET and POST methods and MySQL server.
# Deployed on kubernetes

This repo contains code that 
1) Deploys a Flask API to add, delete and modify users in the MySQL database
2) Deploys a MySQL server on a Kubernetes cluster
2) Attaches a persistent volume to it, so the data remains contained if pods are restarting

## Secrets
`Kubernetes Secrets` can store and manage sensitive information. For this example we will define a password for the
`root` user of the `MySQL` server using the `Opaque` secret type.

1. Encoding password in my terminal: `echo -n super-secret-passwod | base64`
2. Added the output to the `flakapi-secrets.yml` file at the `db_root_password` field

## Deployments
Get the secrets, persistent volume in place and apply the deployments for the `MySQL` database and `Flask API`

1. Added the secrets to my `kubernetes cluster`: `kubectl apply -f flaskapi-secrets.yml`
2. Create the `persistent volume` and `persistent volume claim` for the database: `kubectl apply -f mysql-pv.yml`
3. Create the `MySQL` deployment: `kubectl apply -f mysql-deployment.yml`
4. Create the `Flask API` deployment: `kubectl apply -f flaskapp-deployment.yml`

We can check the status of the pods, services and deployments.

## Creating database and schema
The API can only be used if the proper database and schemas are set. This was done via the terminal.
1. Connect to your `MySQL database` by setting up a temporary pod as a `mysql-client`: 
   `kubectl run -it --rm --image=mysql --restart=Never mysql-client -- mysql --host mysql --password=<super-secret-password>`
   make sure to enter the (decoded) password specified in the `flaskapi-secrets.yml`
2. Create the database and table
   1. `CREATE DATABASE flaskapi;`
    2. `USE flaskapi;`
    3. `CREATE TABLE users(user_id INT PRIMARY KEY AUTO_INCREMENT, user_name VARCHAR(255), user_email VARCHAR(255), user_password VARCHAR(255));`
    
## Expose the API
The API can be accessed by exposing it using minikube: `minikube service flask-service`. This will return a `URL`. If you paste this to your browser you will see the `hello world` message. You can use this `service_URL` to make requests to the `API`

## Start making requests
Now we can use the `API` to `CRUD` your database
1. add a user: `curl -H "Content-Type: application/json" -d '{"name": "<user_name>", "email": "<user_email>", "pwd": "<user_password>"}' <service_URL>/create`
2. get all users: `curl <service_URL>/users`
3. get information of a specific user: `curl <service_URL>/user/<user_id>`
4. delete a user by user_id: `curl -H "Content-Type: application/json" <service_URL>/delete/<user_id>`
5. update a user's information: `curl -H "Content-Type: application/json" -d {"name": "<user_name>", "email": "<user_email>", "pwd": "<user_password>", "user_id": <user_id>} <service_URL>/update`
