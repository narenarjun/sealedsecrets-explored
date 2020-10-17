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
> The latest version of installation yaml can be found in the [releases page](https://github.com/bitnami-labs/sealed-secrets/releases). 


