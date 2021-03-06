# Deploying a Flask API

This is the project starter repo for the fourth course in the [Udacity Full Stack Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004): Server Deployment, Containerization, and Testing.

In this project you will containerize and deploy a Flask API to a Kubernetes cluster using Docker, AWS EKS, CodePipeline, and CodeBuild.

The Flask app that will be used for this project consists of a simple API with three endpoints:

- `GET '/'`: This is a simple health check, which returns the response 'Healthy'.
- `POST '/auth'`: This takes a email and password as json arguments and returns a JWT based on a custom secret.
- `GET '/contents'`: This requires a valid JWT, and returns the un-encrpyted contents of that token.

The app relies on a secret set as the environment variable `JWT_SECRET` to produce a JWT. The built-in Flask server is adequate for local development, but not production, so you will be using the production-ready [Gunicorn](https://gunicorn.org/) server when deploying the app.

## Initial setup
1. Fork this project to your Github account.
2. Locally clone your forked version to begin working on the project.

## Dependencies

- Docker Engine
    - Installation instructions for all OSes can be found [here](https://docs.docker.com/install/).
    - For Mac users, if you have no previous Docker Toolbox installation, you can install Docker Desktop for Mac. If you already have a Docker Toolbox installation, please read [this](https://docs.docker.com/docker-for-mac/docker-toolbox/) before installing.
 - AWS Account
     - You can create an AWS account by signing up [here](https://aws.amazon.com/#).

## Project Steps

Completing the project involves several steps:

1. Run the API locally using the Flask server (no containerization)
- open one terminal and start the Flask server
```
pip install -r requirements.txt
export JWT_SECRET='myjwtsecret'
export LOG_LEVEL=DEBUG
python3 main.py
```
- open another terminal, test the API endpoint (/auth and /contents), replace email and password with some valid string, you should see the email in the decrypted content
```
brew install jq
export TOKEN=`curl -d '{"email":"<EMAIL>","password":"<PASSWORD>"}' -H "Content-Type: application/json" -X POST localhost:8080/auth  | jq -r '.token'`
curl --request GET 'http://127.0.0.1:8080/contents' -H "Authorization: Bearer ${TOKEN}" | jq .
```
2. Write a Dockerfile for a simple Flask API (finish the dockerfile and env_file)
3. Build and test the container locally
- open one terminal and start the Flask server in docker
```
docker build --tag jwt-api-test
docker run  -p 80:8080 --env-file=env_file jwt-api-test
```
- open another terminal, test the API endpoint (/auth and /contents)
```
export TOKEN=`curl -d '{"email":"<EMAIL>","password":"<PASSWORD>"}' -H "Content-Type: application/json" -X POST localhost:80/auth  | jq -r '.token'`
curl --request GET 'http://127.0.0.1:80/contents' -H "Authorization: Bearer ${TOKEN}" | jq .
```
4. Create an EKS cluster
- create a aws iam role
- install awscli, eksctl, and kubectl
```
pip3 install awscli --upgrade
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl
brew install kubectl
```
- create an EKS named simple-jwt-api
```
eksctl create cluster --name simple-jwt-api
eksctl delete cluster --name simple-jwt-api (deleted when necessary)
```
5. Create an IAM Rule that CodeBuild can use to interact with EKS
- set environment variable ACCOUNT_ID
```
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
```
- create a role policy
- create a role 'UdacityFlaskDeployCBKubectlRole'

6. Store a secret using AWS Parameter Store
7. Create a CodePipeline pipeline triggered by GitHub checkins
- Generate a GitHub access token
- update buildspec.yml
- Put secret into AWS Parameter Store
```
aws ssm put-parameter --name JWT_SECRET --value "YourJWTSecret" --type SecureString
```
- update ci-cd-codepipeline.cfn.yml
- Create a stack for CodePipeline in CloudFormation service
- Check the pipeline works in CodePipeline UI
8. Create a CodeBuild stage which will build, test, and deploy your code

For more detail about each of these steps, see the project lesson [here](https://classroom.udacity.com/nanodegrees/nd004/parts/1d842ebf-5b10-4749-9e5e-ef28fe98f173/modules/ac13842f-c841-4c1a-b284-b47899f4613d/lessons/becb2dac-c108-4143-8f6c-11b30413e28d/concepts/092cdb35-28f7-4145-b6e6-6278b8dd7527).
