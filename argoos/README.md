# Argoos addons

Argoos aimes to provide a way to do: "Push image in registry to deploy in Kubernetes".

See: https://github.com/Smile-SA/argoos to get more information.

## Configure Docker registry

Change your private docker registry configuration to use "notification" part:

```bash
docker run --rm --entrypoint=cat registry:2 /etc/docker/registry/config.yml | tee config.yml
```

Edit `config.yml` file and add "notifications" directive:

```yaml
notifications:
  endpoints:
    - name: argoos
      url: http://argoos.kube-system/event
      timeout: 500ms
      threshold: 5
      backoff: 1s
```

If your docker registry cannot contact Kubernetes services by internal name, you can change `url` to point on Argoos service by exposed IP, hostname or exposed port. 

E.g Change Argoos service yaml file to use NodePort instead of ClusterIP and note the opened port. In `config.yaml` you may now use http://NODE.IP:EXPOSED-PORT where "NODE.IP" is one of you cluster node ip and "EXPOSED-PORT" the exposed port by the service (commonly > 32000).

## Deploy Argoos

You can deploy Argoos using deployment and service files from the `deploy` directory.

```bash
cd argoos
kubectl create -f ./deploy
# or use the provided Makefile
make deploy
```

Right now, each Docker "image push" will communicate with Argoos. Argoos will check which Deployment is concerned by that image, check update policy, and start an update if it's ok.

## Deploy with Token protection

Argoos may use token to protect access. The notification directive in prviate docker registry should add that header:


```bash
notifications:
  endpoints:
    - name: argoos
      url: http://argoos.kube-system/event
      headers:
        X-Argoos-Token: TOKEN_TO_SEND
      timeout: 500ms
      threshold: 5
      backoff: 1s
```

Replace `TOKEN_TO_SEND` by a long secret key. It's recommanded to save that key in a "secret object". 

Then, add an environment variable named "TOKEN" in the Argoss deployment. 

If you want to use a secret, you can use the provided makefile that automates that process:

```bash
make deploy-with-token
```

Or you can do it yourself:

```bash
kubectl create secret generic argoos-secrets --from-literal=token=write-a-token-there
```

then add the env part:

```yaml
# ...
containers:
- image: smilelab/argoos:1.0.0
  name: argoss
  env:
  - name: TOKEN
    valueFrom:
      secretKeyRef:
        name: argoos-secrets
        key: token
```

## Use TLS/SSL

Argoss can serve SSL. You should provide private key and certificate as environment variable. As for TOKEN, it's recommanded to save certificates in secret object.

Note that Docker Registry should trust your certificate if you want to provide a self-signed certificate.

You can try using the provided makefile:

```bash
make deploy-secured
```

That command create both private key and certificate, and create a "argoos-certificates" object. Right now, Docker Registry notification url should be "https" and not "http".



