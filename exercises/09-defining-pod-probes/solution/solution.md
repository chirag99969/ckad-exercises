# Solution

Create the intial YAML with the following command.

```shell
$ kubectl run hello --image=cybersecnerd/nodejs-hello-world:default --port=3000 -o yaml --dry-run=client --restart=Never > pod.yaml
```

### Edit the YAML file to add READINESS PROBE.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
spec:
  containers:
  - image: cybersecnerd/nodejs-hello-world:default
    name: hello-world
    ports:
    - name: nodejs-port
      containerPort: 3000
    readinessProbe:
      httpGet:
        path: /
        port: nodejs-port
      initialDelaySeconds: 2
      periodSeconds: 8
```

Create the Pod from the YAML file, shell into the Pod as soon as it is running and execute the `curl` command.

```shell
$ kubectl create -f pod.yaml
pod/hello created
$ kubectl exec hello -it -- /bin/sh
# curl localhost:3000
Hello World
# exit
```

Rendering the logs of the Pod reveals additional log output.

```shell
$ kubectl logs pod/hello
Magic happens on port 3000

$ kubectl describe pod readiness-pod | grep -C 2  Readiness
Ready:          True
    Restart Count:  0
    Readiness:      http-get http://:nodejs-port/ delay=2s timeout=1s period=8s #success=1 #failure=3
    Environment:    <none>
    Mounts:
```

### LIVENESS Probe 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
  - image: busybox
    name: app
    args:
    - /bin/sh
    - -c
    - 'while true; do touch /tmp/heartbeat.txt; sleep 5; done;'
    livenessProbe:
      exec:
        command:
        - test `find /tmp/heartbeat.txt -mmin -1`
      initialDelaySeconds: 5
      periodSeconds: 30
```

```shell
$ kubectl describe pod liveness-pod | grep -C 2 Liveness:

Ready:          True
    Restart Count:  2
    Liveness:       exec [test `find /tmp/heartbeat.txt -mmin -1`] delay=5s timeout=1s period=30s #success=1 #failure=3
    Environment:    <none>
    Mounts:
```

