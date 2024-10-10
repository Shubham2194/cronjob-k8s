# cronjob-k8s

Lets say you have some scripts that need to be run at 9AM IST in your K8S environment.

what is Cron ?
CronJob is meant for performing regular scheduled actions such as backups, report generation, and so on. One CronJob object is like one line of a crontab (cron table) file on a Unix system. It runs a Job periodically on a given schedule, written in Cron format

Step 1: 
Lets create a cluster to do POC , i am using kind here for POC.

kind create cluster --name xyz

![image](https://github.com/user-attachments/assets/fa11906b-f400-4254-b0d0-a05e49a7de25)

Step 2:
Note down the commands you want to run with the cronjob and the schedule.
(SCENERIO: i am cloning my repo here and running script present in my repo at 9AM IST everyday)

```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```
This is the simple cron which is just printing the date and echoing Hello from the Kubernetes cluster

Step 3:
Lets make changes in the script with respect to our scenerio.
Create one namespace with the name cronjob
```sh
kubectl create ns cronjob 
kubens cronjob
```
(kubens is a utility to switch namespaces easily)

* Now we can either try cloning our repo from HTTPS or from SSH through our cronjob yml.
* We are proceeding with HTTPS here, lets clone repo from HTTPS (COPY your github repo HTTPS URL)

Create a Secret which is having our Github-token using imperative way:

```sh
kubectl create secret generic github-token --from-literal=token=YOUR_TOKEN
```


Step 4:
Add github URL in our cron yml and try running it :

```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  template:
    spec:
      containers:
      - name: hello
        image: node:16-alpine # Using Node.js image to handle npm commands
        imagePullPolicy: IfNotPresent
        - name: GITHUB_TOKEN
          valueFrom:
            secretKeyRef:
              name: github-token
              key: token
        command:
        - /bin/sh
        - -c
        - |
          apk add --no-cache git
          git clone -b develop https://$GITHUB_TOKEN@github.com/flipspacesit/vizdom.apis.core.git
      restartPolicy: OnFailure

```
Firstly we are creating a job to test if it working fine then we will we add cron and schedule

Step 5:
Clone my repo and hit

```sh
kubectl apply -f cron.yaml
```

Check the cron status 

```sh
kubectl get jobs --watch
```
![image](https://github.com/user-attachments/assets/18202ea3-5aed-4e19-976a-9a93d42dfb0a)

```sh
kubectl get cronjobs
```
![image](https://github.com/user-attachments/assets/b3f3eefc-6a43-479e-9c26-6fe803510160)


Get the logs of the cron job

```sh
pods=$(kubectl get pods --selector=job-name=my-job -o jsonpath='{.items[*].metadata.name}')
 kubectl logs $pods
```

Step 6:

