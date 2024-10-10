# cronjob-k8s

Lets say you have some scripts that need to be run at 9AM IST in your K8S environment.

what is Cron ?
CronJob is meant for performing regular scheduled actions such as backups, report generation, and so on. One CronJob object is like one line of a crontab (cron table) file on a Unix system. It runs a Job periodically on a given schedule, written in Cron format

#Lets create a cluster to do POC , i am using kind here for POC

kind create cluster --name xyz

