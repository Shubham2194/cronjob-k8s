# Cronjob-k8s

![image](https://github.com/user-attachments/assets/fedf9773-849a-4d80-b061-9c4e06b1341b)



Assume you need to run a Job periodically on a given schedule,that need to be run at 9AM IST in your K8S environment, we can achieve it using CronJobs.

what is a Cronjob ?

A CronJob in Kubernetes is a resource that schedules recurring tasks on a time-based schedule. This is similar to cron tasks in UNIX and are a vital tool for system maintenance and automation.

Step 1: 
Lets create a cluster to do POC , 
i am using kind here for POC.

kind create cluster --name xyz

![image](https://github.com/user-attachments/assets/fa11906b-f400-4254-b0d0-a05e49a7de25)

Step 2:
Note down the commands you Would like to run with the cronjob and the schedule timimngs i.e making backups, creating reports, sending emails, cleanup tasks etc

(Our SCENERIO for this POC : i am going to clone my repo with cronjob and run my bash script present in my repository at 9AM IST everyday)

Create a file and the paste the below content.
```sh
nano cronjob.yaml
```

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
This is the simple cron which is just printing the date and echoing Hello from the Kubernetes cluster.

```sh
kubectl apply -f cronjob.yaml
kubectl get cronjob hello
kubectl get jobs --watch
kubectl get cronjob hello
```

Note:
The job name is different from the pod name.
```sh
pods=$(kubectl get pods --selector=job-name=hello-change_with_yours -o jsonpath='{.items[*].metadata.name}')
kubectl logs $pods
```
The output is similar to this:

Thrus oct 10 11:02:09 UTC 2024
Hello from the Kubernetes cluster


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
        image: node:16-alpine # Updated to Node.js 16.x for better package compatibility
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
          npm install -g npm@latest # Ensure npm is updated
          git clone -b develop https://$GITHUB_TOKEN@github.com/your-repo.git
          pwd
          ls && npm -v
          cd your-repo
          npm install --no-optional
      restartPolicy: OnFailure

```

Firstly we are creating a job to test if it will work fine then we will set the same as cron and add schedule.

Step 5:
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

![image](https://github.com/user-attachments/assets/7cc60b69-dd27-4c48-b384-35b745ef9d7b)

we are successfully able to Clone and Run nun some set of commands using Job.

Step 6:
Let's setup it at 9AM IST with cronjob and run the script which is present in the repo.

Clone my repo and hit
```sh
git clone https://github.com/Shubham2194/cronjob-k8s.git

kubectl apply -f cron.yaml
kubectl get cronjobs
```

Cheers we are able to Setup cron in Kubernetes Successfully !!
