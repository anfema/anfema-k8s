# Configuration for Services in Pods

## Configuration

To create configuration to be accessed from a Pod you need to supply a
`ConfigMap` manifest.

Configuration may be read from within a Pod via environment Variables or mounted
as a File into the Pod.

Usually ConfigMaps are mutable, all changes will be sent to all Pods the instant
they are changed. This puts load on the `etcd` as it has to monitor all accesses
to those variables. If your application needs to be restarted to reload its
config it is advisable to define the configuration as imutable.

Example of a `ConfigMap`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```

You can define key-value items which will be exported as files with any format
you like (`user-interface.properties`) or as plaintext files.
All keys/value items may be exported as environment variables too.

You can create config maps from an existing config file like this:

```bash
kubectl create configmap <name> --from-file=<filename>
```

Which will result in a configmap like the following which will be applied
instantly:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: <name>
data:
  <filename>: |
    <file-contents>
```

Fetch a config map with this command:

```bash
kubectl get configmap <name> -o yaml
```

## Secrets

To create a secret import a manifest with the secret values Base64 encoded to
enable binary secrets and accidental reading of the values.

Secrets work like `ConfigMap` entities basically, but you can configure the
cluster to encrypt the secrets on disk.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

To base64 encode a value run the following command:

```bash
echo -n '<value>' | base64
```

If you want to save a secret in its cleartext-form you absolutely can do that:
just replace `data` with `stringData`. you can mix and match `data` and
`stringData`, but be aware `stringData` overrides `data` if the key is the same.

To insert a secret just apply the manifest:

```bash
kubectl apply -f ./secret.yml
```

you can fetch and edit secrets:

```bash
kubectl get secret <name> -o yaml
kubectl edit secret <name>
```

and you can delete them as well:

```bash
kubectl delete secret <name>
```

## Using secrets and configmaps in containers
