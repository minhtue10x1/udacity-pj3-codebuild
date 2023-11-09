# Install postgres in kubenetes

Configure the Project
Configure a database for the service.
Set up a Postgres database using a Helm Chart.

Set up Bitnami Repo
helm repo add postgresql https://charts.bitnami.com/bitnami
Install PostgreSQL Helm Chart
Run pvc and pv
kubectl apply -f analytics/postgresql-pv.yaml
kubectl apply -f analytics/postgresql-pv-claim.yaml
helm install postgres-svc postgresql/postgresql --set primary.persistence.existingClaim=pvc1 --set volumePermissions.enabled=true

export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgres-svc-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d) echo $POSTGRES_PASSWORD

echo $POSTGRES_PASSWORD
The instructions are adapted from Bitnami's PostgreSQL Helm Chart.

Test Database Connection The database is accessible within the cluster. This means that when you will have some issues connecting to it via your local environment. You can either connect to a pod that has access to the cluster or connect remotely via Port Forwarding
Connecting Via Port Forwarding

kubectl port-forward --namespace default svc/postgres-svc-postgresql 5432:5432 &
PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432
Connecting Via a Pod

kubectl exec -it postgres-svc-postgresql bash
PGPASSWORD="<PASSWORD HERE>" psql postgres://postgres@postgres-svc-postgresql:5432/postgres -c
Run Seed Files We will need to run the seed files in db/ in order to create the tables and populate them with data.
kubectl port-forward --namespace default svc/<SERVICE_NAME>-postgresql 5432:5432 &
PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < db/1_create_tables.sql
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < db/2_seed_users.sql
PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < db/3_seed_tokens.sql
Running the Analytics Application Locally
In the analytics/ directory:

Install dependencies
pip install -r requirements.txt
Run the application (see below regarding environment variables)
<ENV_VARS> python app.py
There are multiple ways to set environment variables in a command. They can be set per session by running export KEY=VAL in the command line or they can be prepended into your command.

DB_USERNAME
DB_PASSWORD
DB_HOST (defaults to 127.0.0.1)
DB_PORT (defaults to 5432)
DB_NAME (defaults to postgres)
If we set the environment variables by prepending them, it would look like the following:

DB_USERNAME=username_here DB_PASSWORD=password_here python app.py
The benefit here is that it's explicitly set. However, note that the DB_PASSWORD value is now recorded in the session's history in plaintext. There are several ways to work around this including setting environment variables in a file and sourcing them in a terminal session.

Verifying The Application
Generate report for check-ins grouped by dates

curl <BASE_URL>/api/reports/daily_usage
Generate report for check-ins grouped by users

curl <BASE_URL>/api/reports/user_visits

# CREATE AWS codebuild

Go in aws codebuild create project build with auto build when push code from github

# CREATE and use AWS ECR

Retrieve an authentication token and authenticate your Docker client to your registry.
Use the AWS CLI:

aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 516661558134.dkr.ecr.us-east-1.amazonaws.com
Note: If you receive an error using the AWS CLI, make sure that you have the latest version of the AWS CLI and Docker installed.
Build your Docker image using the following command. For information on building a Docker file from scratch see the instructions here . You can skip this step if your image is already built:

docker build -t udacity-pj3-cluster .
After the build completes, tag your image so you can push the image to this repository:

docker tag udacity-pj3-cluster:latest 516661558134.dkr.ecr.us-east-1.amazonaws.com/udacity-pj3-cluster:latest
Run the following command to push this image to your newly created AWS repository:

docker push 516661558134.dkr.ecr.us-east-1.amazonaws.com/udacity-pj3-cluster:latest

# CREATE and use AWS EKS

Create cluster with name coworking-prj3 have node-group. node group has 2 node t4g.medium

# Deploy application

first: Run kubectl apply -f analytics/deployment/db-configmap.yaml
kubectl apply -f analytics/deployment/db-secret.yaml
kubectl apply -f analytics/deployment/deployment.yaml
