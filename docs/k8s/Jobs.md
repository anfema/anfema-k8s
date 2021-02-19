# One-off Jobs

You can run one-off Jobs on the cluster to e.g. run a DB migration or clean up
some data.

It works similar to a deployment but uses another `apiVersion` and `kind`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  ttlSecondsAfterFinished: 0
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

Run the job by applying it:

```bash
kubectl apply -f pi.yaml
```

Always add a `ttlSecondsAfterFinished` if you want the garbage left behind by
the job to be automatically cleaned up. It deletes the logs though, so be aware.
If you do not use TTL you have to delete the job after being finished with it:

```bash
kubectl delete jobs/<jobname>
```

If you want a job to have a deadline to which it must have completed or be
deleted and marked failed set `activeDeadlineSeconds` in the `spec`
