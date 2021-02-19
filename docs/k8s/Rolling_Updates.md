# Rolling updates

If your service is replicated you can do a rolling update which starts a new
Pod with the updated image, waits until it becomes ready and then kills one of
the outdated Pods. This goes on until all Pods are replaced.

To run a rollout on a replicated service just update the deployment `YAML` file
and apply it. Make sure that you have `replicas` greater than `1` for it to be
effective. Be aware that you'll have Pods of differing versions running at the
same time.

To see the rollout status run this:

```bash
kubectl rollout status deployment/<deployment-name>
```

If you want kubernetes to wait for a Pod to become ready instead of assuming
that it is ready as soon as it has been started you can add `minReadySeconds` to
your deployment `spec`. Be aware that startup tasks always delay the Pod
becoming ready even if `minReadySeconds` is 0.

## Deployment history

To see the deployment history that is available to be rolled back to run

```bash
kubectl rollout history deployment.v1.apps/<deployment-name>
```

## Rollbacks

If your deployment failed somehow you can do a roll-back:

```bash
kubectl rollout undo deployment.v1.apps/<deployment-name>
```

You can define how many old deployments kubernetes should keep by defining
`revisionHistoryLimit` in your deployment `spec`. The default is 10.
