1. Command: kind create cluster
    - kubectl cluster-info --context kind-kind
       
2. kubectl create namespace julian
    - kubectl get namespaces
    - kubectl config set-context --current --namespace=julian
    - kubectl config view --minify | grep namespace:


3. 
- Command: kubectl apply -f deployment.yaml

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: julian
    labels:
        created-by: julian
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: julian
    template:
        metadata:
        labels:
            app: julian
            created-by: julian
        spec:
        containers:
            - name: http-https-echo
            image: mendhak/http-https-echo
            env:
                - name: ECHO_INCLUDE_ENV_VARS
                value: "1"
            livenessProbe:
                httpGet:
                path: /
                port: 8080
                initialDelaySeconds: 5
                periodSeconds: 10
            readinessProbe:
                httpGet:
                path: /
                port: 8080
                initialDelaySeconds: 5
                periodSeconds: 10
    ```

- kubectl get deployments
    ```
    NAME     READY   UP-TO-DATE   AVAILABLE   AGE
    julian   0/1     1            0           23s
    ```
- kubectl get pods
    ```
    NAME                     READY   STATUS    RESTARTS   AGE
    julian-8848cd65b-qnj52   0/1     Running   0          4s 
    ```

   
4. kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml
 
    - kubectl get pods -n kube-system
    ```
    NAME                                         READY   STATUS    RESTARTS      AGE
    coredns-7c65d6cfc9-rm8l8                     1/1     Running   1 (11m ago)   3h59m
    coredns-7c65d6cfc9-wj44m                     1/1     Running   1 (11m ago)   3h59m
    etcd-kind-control-plane                      1/1     Running   1 (11m ago)   3h59m
    kindnet-vp55g                                1/1     Running   1 (11m ago)   3h59m
    kube-apiserver-kind-control-plane            1/1     Running   1 (11m ago)   3h59m
    kube-controller-manager-kind-control-plane   1/1     Running   1 (11m ago)   3h59m
    kube-proxy-l2fcx                             1/1     Running   1 (11m ago)   3h59m
    kube-scheduler-kind-control-plane            1/1     Running   1 (11m ago)   3h59m
    metrics-server-bc86b778d-lp8bl               0/1     Running   0             21s
    ```

- kubectl patch -n kube-system deployment metrics-server --type=json -p '[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
    - kubectl get deployment metrics-server -n kube-system

    ```
    NAME             READY   UP-TO-DATE   AVAILABLE   AGE
    metrics-server   1/1     1            1           2m23s
    ```

5. kubectl top pod
    ```
    NAME                      CPU(cores)   MEMORY(bytes)   
    julian-5996d97c48-b5cpx   1m           21Mi     
    ```
  - New deployment
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: julian
    labels:
        created-by: julian
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: julian
    template:
        metadata:
        labels:
            app: julian
            created-by: julian
        spec:
        containers:
            - name: http-https-echo
            image: mendhak/http-https-echo
            env:
                - name: ECHO_INCLUDE_ENV_VARS
                value: "1"
            livenessProbe:
                httpGet:
                path: /
                port: 8080
                initialDelaySeconds: 15
                periodSeconds: 10
            readinessProbe:
                httpGet:
                path: /
                port: 8080
                initialDelaySeconds: 15
                periodSeconds: 10
            resources:
                requests:
                memory: "21Mi"
                cpu: "1m"
                limits:
                memory: "32Mi"
                cpu: "10m"

    ```


  Why These Values?
- CPU: Request (1m): Based on actual usage, this is sufficient for the pod’s normal operation.
- Limit (10m): Allows some room for bursts but prevents excessive CPU usage.
Memory:
- Request (21Mi): Matches the observed memory usage.
- Limit (32Mi): Provides a reasonable buffer in case the pod needs a bit more memory.




<br>

- kubectl apply -f deployment.yaml  


Nachdem mit diesen Zahlen ständig ein neuer Pod hochgefahren wurde, habe ich die Zahlen erhöht und Puffer eingebaut:

```yaml
resources:
    requests:
    memory: "40Mi"
    cpu: "20m"
    limits:
    memory: "64Mi"
    cpu: "100m"
```
<br>

6. Two replicas
    ```yaml
    ...
    spec:
        replicas: 2
    ...
    ```
- kubectl get deployments
```
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
julian   2/2     2            2           4h10m
```


- kubectl get pods

```
NAME                      READY   STATUS    RESTARTS   AGE
julian-84975c8ddf-4vh2l   1/1     Running   0          2m27s
julian-84975c8ddf-jvkr8   1/1     Running   0          9m22s
```

7. 
    service.yaml
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
    name: julian
    labels:
        created-by: julian
    spec:
    selector:
        app: julian
    ports:
        - protocol: TCP
        port: 8000      
        targetPort: 8080 
    type: ClusterIP

    ```
- kubectl apply -f service.yaml
- kubectl get svc
    ```
    NAME     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
    julian   ClusterIP   10.96.215.24   <none>        8000/TCP   84s
    ```
8. kubectl port-forward svc/julian 8000:8000
- curl http://localhost:8000
```
{
  "path": "/",
  "headers": {
    "host": "localhost:8000",
    "user-agent": "curl/8.4.0",
    "accept": "*/*"
  },
  "method": "GET",
  "body": "",
  "fresh": false,
  "hostname": "localhost",
  "ip": "::ffff:127.0.0.1",
  "ips": [],
  "protocol": "http",
  "query": {},
  "subdomains": [],
  "xhr": false,
  "os": {
    "hostname": "julian-84975c8ddf-jvkr8"
  },
  "connection": {},
  "env": {
    "KUBERNETES_PORT": "tcp://10.96.0.1:443",
    "KUBERNETES_SERVICE_PORT": "443",
    "NODE_VERSION": "18.20.4",
    "HOSTNAME": "julian-84975c8ddf-jvkr8",
    "YARN_VERSION": "1.22.19",
    "SHLVL": "1",
    "HOME": "/home/node",
    "ECHO_INCLUDE_ENV_VARS": "1",
    "KUBERNETES_PORT_443_TCP_ADDR": "10.96.0.1",
    "HTTP_PORT": "8080",
    "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
    "KUBERNETES_PORT_443_TCP_PORT": "443",
    "KUBERNETES_PORT_443_TCP_PROTO": "tcp",
    "HTTPS_PORT": "8443",
    "KUBERNETES_PORT_443_TCP": "tcp://10.96.0.1:443",
    "KUBERNETES_SERVICE_PORT_HTTPS": "443",
    "KUBERNETES_SERVICE_HOST": "10.96.0.1",
    "PWD": "/app"
  }
}%
```

9. Es gibt kein "--watch" beim "top pod" Befehl, deswegen habe ich es so gemacht: 
    ``` 
    while true; do kubectl top pod -n julian >> kubectl_top_output.txt; sleep 1; done
    ```
    Die Requests habe ich so gemacht:
    ```
    for i in {1..1000}; do curl -s http://localhost:8000 > /dev/null; done
    ```
    Ein Beispielergebnis:
    ```
    NAME                      CPU(cores)   MEMORY(bytes)   
    julian-84975c8ddf-4vh2l   2m           25Mi            
    julian-84975c8ddf-jvkr8   94m          45Mi 
    ```
    --> Der erste Pod wurde nie verwendet, weil das eingestellte Limit hoch genug war. Deswegen habe ich das Limit etwas gesenkt, um die Last auf zwei Pods zu verteilen:
    
    ```yaml
    resources:
        requests:
            memory: "40Mi"
            cpu: "20m"
        limits:
            memory: "48Mi"  
            cpu: "50m"     
    ```

    Außerdem soll es insgesamt mindestens eine Minute dauern, deswegen baue ich ein sleep 0.06 ein:
    ```
    for i in {1..1000}; do
        curl -s http://localhost:8000 > /dev/null
        sleep 0.06 
    done
    ```
- Jetzt war die maximale Auslastung wie folgt:
```
NAME                      CPU(cores)   MEMORY(bytes)   
julian-5c4676484c-cp67r   51m          34Mi            
julian-5c4676484c-zrzsd   2m           24Mi   
```
Vermutlich ist sie jetzt geringer, da nach jedem Request 0.06 Sekunden gewartet wird, bis der nächste gemacht wird.
=> Limit etwas verringern