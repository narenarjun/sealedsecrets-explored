# Sealed Secrets Explored 🚀🚀🚀🚀

[Sealed Secrets](https://github.com/bitnami/sealed-secrets) are a "one-way" encrypted Secret that can be created by anyone, but can only be decrypted by the controller running in the target cluster. **The Sealed Secret is safe to share publicly**, upload to git repositories, post to twitter, etc. Once the SealedSecret is safely uploaded to the target Kubernetes cluster, the sealed secrets controller will decrypt it and recover the original Secret.

Sealed Secrets is composed of two parts:

- cluster-side controller / operator
- client-side utility: kubeseal

⭐ Get the latest [kubeseal cli](https://github.com/bitnami-labs/sealed-secrets/releases) binary for your platform and add it to your path.

## Install SealedSecrets to your cluster

```bash
kubectl apply -f ./install/sealedsecret-controller-install.yaml
```

> ### 📓 Note:
>
> The latest version of installation yaml can be found in the [releases page](https://github.com/bitnami-labs/sealed-secrets/releases).

## Let's explore 🛠

### ⛓ Creating few namespaces:

Lets create 2 namespaces such as `play1` and `play2`.

imperative way:

```bash
kubectl create ns play1
kubectl create ns paly2
```

or

declarative way:

```bash
kubectl apply -f ./k8s/namespaces/ns-play1.yaml
kubectl apply -f ./k8s/namespaces/ns-play2.yaml
```

Let's move to the `play1` namespace.

```bash
kubens play1
```

> ### 📚 Note
>
> [kubens and kubectx](https://github.com/ahmetb/kubectx) are awesome and a must have k8s cli tool.
> Get them from [here](https://github.com/ahmetb/kubectx).

### 🚧 Lets Generate secrets and SealedSecrets:

Let's create a avengers level secret.

```bash
kubectl create secret generic avengers --dry-run=client --from-literal=strongest="The strongest Avanger is Hulk"  -o yaml > avengers-secret.yaml
```

output in the [`avengers-secret.yaml`](./secrets/avengers-secret.yaml):

```yaml
apiVersion: v1
data:
  strongest: VGhlIHN0cm9uZ2VzdCBBdmFuZ2VyIGlzIEh1bGs=
kind: Secret
metadata:
  creationTimestamp: null
  name: avengers
```

here we didn't specify any namespace scope, we get this yaml.

we can easily decode the data since it's only base64,

```bash
echo VGhlIHN0cm9uZ2VzdCBBdmFuZ2VyIGlzIEh1bGs= | base64 --decode
```

it's not that hard to decode:

```bash
The strongest Avanger is Hulk
```

now it's time to seal the secret.

using `kubeseal` cli , we can generate the sealedsecret.

```bash
kubeseal --format yaml <avengers-secret.yaml >sealsrt1.yaml
```

the output in the [`sealsrt1.yaml`](./sealedsecrets/sealsrt1.yaml) :

```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  creationTimestamp: null
  name: avengers
  namespace: play1
spec:
  encryptedData:
    strongest: AgA8dHOzqEtaepgei19m3ENVKLSzjJ6r1xdmYrsc+hTl7tYK0C9k/TRivTOPecwEyPZz2yQdRV8pWaKLIbaave6hAo5D6XosiQK+WiI7tWmYtBcOwDQKzonlrmnRYz0KByAJc7kMXagLpAqNwmnjyrBBhZU3qgVheA451vay5Xz0H9BRXAfp7YHNLZkU8uZr2NVwztvdKrLWjrLsRii63zQW0y4VmThRsj5eyttTX+s2/b27oKS1ZvcgYyloILB8zL3KYUi4sUdQRmwEGq4EZM8HqkaA85o0g7w/VZjjZY60mQXoyZ+MMFLZhiEBTWlW/0g8jZaaXm2KvWEgfRd5cOn0NsrEbVvsUnIvLzZilOrxuTohWZl6m5WfZZ4Z7HV8nGnKQDZD/YrnJrZyRQiH4FC85WUnOBh6n0X6L/fH/6waJqyhl4TWfavhBbnti9k61bouDRL2s/DeVEEOo/bZJfBIquI7CE7bvVD5n1EK75Rp57cp3sB3BNh0YWmPy7UOo2fCKBHyWy8CMjojNvUEpVmyQSl+hbNs0aV12cBw5VYWTzoQeICheeuktBeT8cRNOQcat5mIFUEng2c8iZee9mC18D7eyjjtDjptx8aFJ0iJdEbl+UGzRj4HOjbYxiAOEIrtynnDd4MiNSM2yVhVx43kdlYpxtStUeh2PlimmSbob+7J6G2Pa3mPbO+mOWN1EIiEdj15J0KQY9wnsxAO40snnA4dWxG2ICXSb0GiNw==
  template:
    metadata:
      creationTimestamp: null
      name: avengers
      namespace: play1
```

we got a namespace specified in the sealedsecret yaml, but didn't mention namespace in the secret yaml or during sealsecret creation, so how it's persent in the yaml ?

This comes from the `scope` of `SealedSecrets`.

There are three scopes you can create our `SealedSecrets` with:

- `strict (default)`: In this case, you need to seal your Secret considering the name and the namespace. You can’t change the name and the namespaces of your SealedSecret once you've created it. If you try to do that, you get a decryption error.

- `namespace-wide`: This scope allows you to freely rename the SealedSecret within the namespace for which you’ve sealed the Secret.

- `cluster-wide`: This scope allows you to freely move the Secret to any namespace and give it any name you wish.

We can select the scope with the `--scope` flag while using `kubeseal`:

```bash
$ kubeseal --scope cluster-wide --format yaml <secret.yaml >sealed-secret.yaml
```

> ### 📚 Note:
>
> when not `--scope` is given it defaults to `strict`

We can also use annotations within our `Secret` to apply scopes before we pass the configuration to `kubeseal`:

```yaml
sealedsecrets.bitnami.com/namespace-wide: "true" #for namespace-wide
sealedsecrets.bitnami.com/cluster-wide: "true" #for cluster-wide
```

> ### 📚 Note:
>
> If we don’t specify any `annotations`, then kubeseal assumes a strict scope. If we set both annotations, `cluster-wide` takes precedence.

Let's apply our avengers sealedscret to the namespace.

```bash
$>> kubectl apply -f ./sealedsecrets/sealsrt1.yaml
sealedsecret.bitnami.com/avengers created
```

Let's see out secret in the cluster:

```bash
$>> kubectl get secret

NAME        TYPE         DATA    AGE
avengers    Opaque        1      92s
```

### Let's consume the secret:

Apply the pod to the cluster.

```bash
 kubectl apply -f ./k8s/pods/superpod.yaml
```

get the pod:

```bash
>>$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   0          6s
```

Let's print the secret value:

```bash
>>$ kubectl exec -it busybox -- cat /tmp/avengersecret/strongest
The strongest Avanger is Hulk
```

### Modifying Namespace or SecretName

Since we created the `avengers` secret in strict mode , if we modifiy the `namespace` or the `secretname` and apply it to the cluster , the operator will not convert it into secret in the cluster because it was `strict` scoped.

### Creating Cluster-Wide `SealedSecrets`

Let’s create a Secret without specifying the namespace.

```bash
 kubectl create secret generic dragonball --dry-run=client --from-literal=goku="True super saiyan" -o yaml > dbzsecret.yaml
```

Seal the Secret using the cluster-wide scope,

```bash
$ kubeseal --format yaml --scope cluster-wide <dbzsecret.yaml >dbzsealedsecret.yaml
```

and apply it to the `play1` namespace.

```bash
$ kubectl apply -n play1 -f ./sealedsecrets/dbzsealedsecret.yaml
```

Let’s see if the Secret is created.

```bash
kubectl get secret -n play1 dragonball
NAME         TYPE     DATA   AGE
dragonball   Opaque   1      22s

```

Now let’s rename the Secret, and apply it to the `play1` namespace.

```bash
>>$ cp -a dbzsealedsecret.yaml saiyansecret.yaml
```

```bash
>>$ sed -i 's/dragonball/saiyan/g' saiyansecret.yaml
```

```bash
>>$ kubectl apply -f saiyansecret.yaml -n play1
sealedsecret.bitnami.com/saiyan created
```

When we get the Secret, we see that it’s available.

```bash
kubectl get secret saiyan -n play1
NAME     TYPE     DATA   AGE
saiyan   Opaque   1      64s
```

What about modifying the namespace? Let’s apply the sealed Secret on the database namespace.

```bash
>>$ kubectl apply -f saiyansecret.yaml -n play2
sealedsecret.bitnami.com/saiyan created
```

When we get the Secret, we see that it’s present as well.

```bash
kubectl get secret saiyan -n play2
NAME     TYPE     DATA   AGE
saiyan   Opaque   1      7s
```
# ⭐🎉🎉🎉`SealedSecrets` is the `Gitops` way of managing secrets and it's never been eaiser than now. 