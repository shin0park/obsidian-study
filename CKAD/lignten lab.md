A pod called `dev-pod-dind-878516` has been deployed in the default namespace. Inspect the logs for the container called `log-x` and redirect the warnings to `/opt/dind-878516_logs.txt` on the `controlplane` node

kubectl logs dev-pod-dind-878516 -c log-x | grep WARNING > /opt/dind-878516_logs.txt


Create a cronjob called `dice` that runs every `one minute`. Use the Pod template located at `/root/throw-a-dice`. The image `throw-dice` randomly returns a value between 1 and 6. The result of 6 is considered `success` and all others are `failure`.  
  
The job should be `non-parallel` and complete the task `once`. Use a `backoffLimit` of `25`.  
  
If the task is not completed within `20 seconds` the job should fail and pods should be terminated.

You don't have to wait for the job completion. As long as the cronjob has been created as per the requirements.

```
apiVersion: batch/v1 
kind: CronJob metadata: 
name: dice spec: 
schedule: "*/1 * * * *" 
jobTemplate: 
	spec: 
		completions: 1 
		backoffLimit: 25 
		# This is so the job does not quit before it succeeds. 
		activeDeadlineSeconds: 20 
		template: 
			spec: 
			containers: 
			- name: dice 
			  image: kodekloud/throw-dice 
			  restartPolicy: Never
```


We have deployed a few pods in this cluster in various namespaces. Inspect them and identify the pod which is not in a `Ready` state. Troubleshoot and fix the issue.

Next, add a check to restart the container on the same pod if the command `ls /var/www/html/file_check` fails. This check should start after a delay of `10 seconds` and run `every 60 seconds`.

You may delete and recreate the object. Ignore the warnings from the probe.

![[../resources/Pasted image 20230619104430.png|333]]
![[../resources/Pasted image 20230619104443.png|333]]


### Assigning Pods to Nodes
![[../resources/Pasted image 20230619114526.png|222]]