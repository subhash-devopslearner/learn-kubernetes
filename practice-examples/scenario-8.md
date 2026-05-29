# Scenario 8 — Persistent Volume + CronJob
This combines two concepts together:
In staging namespace create a file backup-job.yaml with:

1. PersistentVolumeClaim called backup-pvc 
    - storage 500Mi
    - accessMode ReadWriteOnce

2. CronJob called backup-job that:
    - Runs every minute
    - Image: busybox
    - Mounts backup-pvc at /backup
    - Command: writes current date to /backup/backup.log like this:

        - sh   echo "Backup at $(date)" >> /backup/backup.log
        - restartPolicy: Never

Then verify:

1. kubectl get pvc -n staging
2. kubectl get cronjob -n staging
3. Wait 2 minutes then check the log file:
   - kubectl exec -it <any-backup-pod> -n staging -- cat /backup/backup.log  
    ```You should see multiple backup entries — proving data persists across pod runs!```

## Solution

```
root@controlplane:~$ kubectl create namespace staging
namespace/staging created

root@controlplane:~$ nano backup-job.yaml

root@controlplane:~$ kubectl apply -f backup-job.yaml 
persistentvolumeclaim/backup-pvc created
cronjob.batch/backup-job created

root@controlplane:~$ kubectl get pvc -n staging 
NAME         STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
backup-pvc   Pending                                      local-path     <unset>                 27s

root@controlplane:~$ kubectl get cronjobs.batch -n staging 
NAME         SCHEDULE    TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
backup-job   * * * * *   <none>     False     0        15s             52s

root@controlplane:~$ kubectl get pods -n staging -w
NAME                        READY   STATUS      RESTARTS   AGE
backup-job-29667942-bth52   0/1     Completed   0          67s
backup-job-29667943-8k46n   0/1     Completed   0          7s

^Croot@controlplane:~$ kubectl exec -it backup-job-29667943-8k46n  staging -- cat /backup/backup.log
error: cannot exec into a container in a completed pod; current phase is Succeeded

root@controlplane:~$ kubectl run pvc-inspector --rm -i --tty --image=busybox --namespace=staging --restart=Never --overrides='
{
  "spec": {
    "volumes": [{
      "name": "shared-storage",
      "persistentVolumeClaim": { "claimName": "backup-pvc" }
    }],
    "containers": [{
      "name": "inspector",
      "image": "busybox",
      "command": ["/bin/sh"],
      "volumeMounts": [{ "name": "shared-storage", "mountPath": "/backup" }],
      "stdin": true,
      "tty": true
    }]
  }
}'
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
/ # cat /backup/backup.log 
Backup at Fri May 29 17:42:08 UTC 2026
Backup at Fri May 29 17:43:00 UTC 2026
Backup at Fri May 29 17:44:00 UTC 2026
Backup at Fri May 29 17:45:00 UTC 2026
/ # exit
pod "pvc-inspector" deleted from staging namespace

root@controlplane:~$ kubectl get pods -n staging -w
NAME                        READY   STATUS      RESTARTS   AGE
backup-job-29667943-8k46n   0/1     Completed   0          2m35s
backup-job-29667944-5xntp   0/1     Completed   0          95s
backup-job-29667945-vsv7p   0/1     Completed   0          35s
backup-job-29667946-f92gm   0/1     Pending     0          0s
backup-job-29667946-f92gm   0/1     Pending     0          0s
backup-job-29667946-f92gm   0/1     ContainerCreating   0          0s
backup-job-29667946-f92gm   0/1     Completed           0          1s
backup-job-29667946-f92gm   0/1     Completed           0          2s
backup-job-29667946-f92gm   0/1     Completed           0          3s
backup-job-29667943-8k46n   0/1     Completed           0          3m3s
backup-job-29667943-8k46n   0/1     Completed           0          3m3s
^Croot@controlplane:~$ 
```

