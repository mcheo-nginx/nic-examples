NGINX Plus includes a real‑time activity monitoring interface that provides key load and performance metrics, we refer it as NGINX Dashboard.

https://www.nginx.com/products/nginx/live-activity-monitoring/

For quick demo: you may visit https://demo.nginx.com/ 

In Kubernetes deployment, when we deploy NGINX Plus as NGINX Ingress Controller (NIC), by default this dashboard is enable on port 8080 but is only accessible locally. In order for us to browse this NIC dashboard in our laptop, we will do port-forward from our laptop with the following command:
```
kubectl port-forward <nic pod> -n nginx-ingress 8080:8080
```

NOTE: It is good practise not to expose this dashboard to prevent unauthorized user access.

However, if you still desire to make this NIC dashboard accessible via NIC itself. Here is some idea of what you can do.

* Create secret to contain user credentials

Guide: https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/

In this repo, I have example nic-auth-users file contains user1 and user2, both with "password" as password. 

```
kubectl create secret generic nic-auth-users --from-file=nic-auth-users -n nginx-ingress

```

* Configure NIC with additional arguments
    -enable-snippets        (This is required to configure basic auth)
    -nginx-status-allow-cidrs  (This is required to allow external access. Ideally, we want to lock down to NIC pod IP range)

Example of code in NIC deployment manifest args section:
```
          - -nginx-status-allow-cidrs=10.0.0.0/8
          - -enable-snippets
```

* Mount secret as volume into NIC
Example of code in NIC deployment manifest:

```
        volumeMounts:
        - name: nic-auth-users
          mountPath: "/etc/nginx/.nic-auth-users"
          readOnly: true
      volumes:
      - name: nic-auth-users
        secret:
          secretName: nic-auth-users
```

Redeploy your NIC
```
kubectl apply -f <nic deployment>.yaml -n nginx-ingress
```

* Create service for NIC dashboard
```
kubectl apply -f nic-svc.yaml
```
Take note: When you have multiple NIC deployed, you will lose control on which NIC dashboard to access via this method. Alternatively, you have to update the service manifest to narrow down that "selector" to pin point to specific NIC.

* Create Virtual Server resource for NIC dashboard
```
kubectl apply -f nic-vs.yaml
```



