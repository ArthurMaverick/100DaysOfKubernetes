### One CronJob object is like one line of a crontab (cron table) file. It runs a job periodically on a given schedule, written in Cron format.


Example:


```
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
            image: busybox
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

### Run the example CronJob by using this command:
```
kubectl create -f https://k8s.io/examples/application/job/cronjob.yaml
```
### The output is similar to this:
```
cronjob.batch/hello created
```
### After creating the cron job, get its status using this command:
```
kubectl get cronjob hello
```

###  delete cronjob
```
 delete cronjob
```

```

## Cron schedule syntax:

# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * *
```

## Starting Deadline

The **.spec.startingDeadlineSeconds** field is optional. It stands for the deadline in seconds for starting the job if it misses its scheduled time for any reason. After the deadline, the cron job does not start the job. Jobs that do not meet their deadline in this way count as failed jobs. If this field is not specified, the jobs have no deadline.

If the **.spec.startingDeadlineSeconds** field is set (not null), the CronJob controller measures the time between when a job is expected to be created and now. If the difference is higher than that limit, it will skip this execution.

For example, if it is set to 200, it allows a job to be created for up to 200 seconds after the actual schedule.

## Concurrency Policy
The **.spec.concurrencyPolicy** field is also optional. It specifies how to treat concurrent executions of a job that is created by this cron job. The spec may specify only one of the following concurrency policies:

- Allow (default): The cron job allows concurrently running jobs

- Forbid: The cron job does not allow concurrent runs; if it is time for a new job run and the previous job run hasn't finished yet, the cron job skips the new job run

- Replace: If it is time for a new job run and the previous job run hasn't finished yet, the cron job replaces the currently running job run with a new job run
Note that concurrency policy only applies to the jobs created by the same cron job. If there are multiple cron jobs, their respective jobs are always allowed to run concurrently.


## Suspend
The **.spec.suspend** field is also optional. If it is set to true, all subsequent executions are suspended. This setting does not apply to already started executions. Defaults to false.

## Jobs History Limits
The **.spec.successfulJobsHistoryLimit** and **.spec.failedJobsHistoryLimit** fields are optional. These fields specify how many completed and failed jobs should be kept. By default, they are set to 3 and 1 respectively. Setting a limit to 0 corresponds to keeping none of the corresponding kind of jobs after they finish.