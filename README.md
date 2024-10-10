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
* SSH is more secure , lets clone repo from SSH (COPY your github repo SSH URL)

Create a Secret which is having our ssh private key using imperative way:

```sh
kubectl create secret generic my-ssh-secret --from-file=ssh-private-key=/Users/your-user/.ssh/id_ed25519
```
(if you dont have ssh key present in your local , create one with 
ssh-keygen )

Step 4:
Add github URL in our cron yml and try running it :

```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: sales-sheet-cron
spec:
  schedule: "* * * * *" # Runs every minute for testing. Adjust as needed.
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: node:16-alpine # Using Node.js image to handle npm commands
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - |
              apk add --no-cache openssh # Install SSH client
              mkdir -p ~/.ssh
              echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
              chmod 600 ~/.ssh/id_rsa
              ssh-keyscan -H github.com >> ~/.ssh/known_hosts
              git clone -b develop git@github.com:your-Github_URL.git
          restartPolicy: OnFailure
          # Add SSH private key as a Kubernetes Secret
          env:
          - name: SSH_PRIVATE_KEY
            valueFrom:
              secretKeyRef:
                name: my-ssh-secret
                key: ssh-private-key
```
This will use our private key and add in the known host.

