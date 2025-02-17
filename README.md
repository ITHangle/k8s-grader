# k8s-grader
Automatic Grading Tools for k8s exercises


## Setup

### Install the latest AWS SAM.
```
./aws_sam_setup.sh
```
### Configure AWS Credentials 
Method 1: Configure AWS Account for AWS Academy Learner Lab 
```
export AWS_ACCESS_KEY_ID=ASIAIOSFODNN7EXAMPLE
export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
export AWS_SESSION_TOKEN=AQoDYXdzEJr...<remainder of session token>
export AWS_DEFAULT_REGION=us-east-1
```
Method 2: Set the token with CLI
```
aws configure
aws configure set aws_session_token <Session Token>
```

## Deploy the SAM project
```
cd k8s-grader-api
sam build
sam deploy
```

## Deploy Minikube with CloudFormation
After configure the AWS Credentials, then run.
```
cd k8s/minikube
aws cloudformation create-stack --stack-name minikube-stack --template-body file://minikube.yaml
```
Describe Stack
```
aws cloudformation describe-stacks --stack-name minikube-stack --query 'Stacks[0].Outputs'
```

Delete Minikube
```
aws cloudformation delete-stack --stack-name minikube-stack
```


## Minikube dashboard
For more information on how to make the Minikube dashboard accessible on all IPs (0.0.0.0), refer to [this link](https://unix.stackexchange.com/questions/621369/how-can-i-make-the-minikube-dashboard-answer-on-all-ips-0-0-0-0).

```
kubectl proxy --address=0.0.0.0 --accept-hosts='.*'
minikube dashboard --url
```
**To make the API call working, you must start the kubectl proxy!**

Sample Data
```
{
 "email": "cywong@vtc.edu.hk",
 "client_certificate": "-----BEGIN RSA PRIVATE KEY-----XXX-----END RSA PRIVATE KEY-----\n",
 "client_key": "-----BEGIN CERTIFICATE-----XX-----END CERTIFICATE-----\n",
 "endpoint": "http://3.90.40.12:8001"
}
```
**Please note that is http but not https!**

## Get the minikube key
1. Download labsuser.pem from AWS Academy Learner Lab
2. Upload to k8s/minikube
3. Open Terminal and run ```chmod 400 labsuser.pem```
4. Run 
```
cd k8s/minikube
./download_key.sh
```


## Local Development

To enable auto-completion 
1. Run ```./create_virtural_env.sh```
2. Set Python Interpreter to ```./venv/bin/python```

Install Kubectl command tools for Unit Test
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

1. Run ```./check_minikube_status.sh``` to ensure minikube us running.
2. Run ```./run_kube_proxy.sh ``` to start kube proxy for remote connection.


### Run Local Lambda test
For the first time, generate the env.json.
```
cd k8s-grader-api/events
python set_env.py
```
Test Lambda
```
sam build && sam local invoke GameTaskFunction --event events/event.json --env-vars events/env.json
sam build && sam local invoke GraderFunction --event events/event.json --env-vars events/env.json
```
Test Web API
```
sam build && sam local start-api --log-file log.txt --warm-containers LAZY --env-vars events/env.json
```

API call sequence
1. Register k8s account http://127.0.0.1:3000/save-k8s-account
2. Get Game Task http://127.0.0.1:3000/game-task?email=cywong@vtc.edu.hk&game=game01
3. Check setup is ready http://127.0.0.1:3000/grader?email=cywong@vtc.edu.hk&game=game01&phrase=ready
4. Work on the answer.
5. Run challenge http://127.0.0.1:3000/grader?email=cywong@vtc.edu.hk&game=game01&phrase=challenge
6. Check Result http://127.0.0.1:3000/grader?email=cywong@vtc.edu.hk&game=game01&phrase=check
7. Go back to 2. until it reply "All tasks are completed!"

