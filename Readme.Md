Name:Ali Imran Bin Shahrin

Q1:
Create a deployment named nginx in the qq2 namespace using the nginx image with 3 replicas.


apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: qq2
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80


Q2:
Create a new  deployment name lab-deployment in the qq2 namespace. With one replica in the manifest then scale the deployment to 4 replicas.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: lab-deployment
  namespace: qq2
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

kubectl scale deployment/lab-deployment --replicas=4


Q3:
Create a deployment names nginx-deployment in the qq3 namespace. Perform a rolling update to the deployment by setting the image to nginx:alpine.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: qq3
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

kubectl set image deployment/nginx-deployment nginx=nginx:alpine

Q4
Create ConfigMap name it q5 and Secret name credentials in the default namespace.

The secret file should contain username and password

and the config map 

does not need to have any certain data but it should be functional


Configmap yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: q5
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true    


Secret yaml

apiVersion: v1
kind: Secret
metadata:
  name: credentials
type: Opaque
data:
  username: YWRtaW4=
  password: YmlyZHNhcmVudHJlYWw=

Create a pod named apache using the httpd image in the default namespace containing the following:

Environment variable named CONTENT containing the key content from the  q5   ConfigMap Environment variable named USERNAME containing the key username from the credentials Secret Environment variable named PASSWORD containing the key password from the credentials Secret

apiVersion: v1
kind: Pod
metadata:
  name: apache
spec:
  containers:
  - name: nginx
    image: httpd:latest
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: credentials
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: credentials
          key: password
    - name: CONTENT
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: special-config
              # Specify the key associated with the value
              key: special.how
    ports:
    - containerPort: 80

Q:5
Create a pod named limit using the redis image in the qq3 namespace with the following resource limits and requests:

RequestsCPU: 0.5Memory: 500MiLimitsCPU: 1Memory: 1Gi

apiVersion: v1
kind: Pod
metadata:
  name: limit
spec:
  containers:
  - name: app
    image: redis:latest
    resources:
      requests:
        memory: "500Mi"
        cpu: "500m"
      limits:
        memory: "1Gi"
        cpu: "1000m"


Q6:
Create a Deployment named web in the ca1 namespace with an image of nginx:latest. Create a new Service named web-svc that sits in front of this deployment. Expose the new service publicly by creating an Ingress resource named web-app. The Ingress's host field must be set to the host key value stored in an existing ConfigMap named webapp-host-fqdn, also located in the ca1 namespace. All changes should be performed in the ca1 namespace. The following settings must also be used for the Ingress resource:

Name: webAnnotations: nginx.ingress.kubernetes.io/rewrite-target: /path: /pathType: Prefix


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webAnnotations
  namespace: ca1
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: example.com
    http:
      paths:
      - pathType: Prefix
        path: /path
        backend:
          service:
            name: web-svc
            port:
              number: 80


apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-host-fqdn
  namespace: ca1
data:
  host: example.com  



apiVersion: v1
kind: Service
metadata:
  name: web-svc
  namespace: ca1
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/name: web
  ports:
    - port: 80
      # By default and for convenience, the `targetPort` is set to
      # the same value as the `port` field.
      targetPort: 80

     
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: ca1
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80


Q7:

Execute the following requirements in the prod namespace. Create a new Deployment named app01 using the nginx:latest container image and have it listen on port 80. The deployment should consist of 2 replicas. Create a new Service named app01-svc and expose the newly created deployment using a NodePort. Confirm that you can access the nginx home (index) page using the NodePort service.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: app01
  namespace: prod
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

--- 

apiVersion: v1
kind: Service
metadata:
  name: app01-svc
  namespace: app01 
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: nginx
  ports:
    - port: 80
      # By default and for convenience, the `targetPort` is set to
      # the same value as the `port` field.
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane
      # will allocate a port from a range (default: 30000-32767)
      nodePort: 30007


Q8:
Create a new Deployment named cloud-app01 in the ca2 namespace consisting of 2 replicas. The deployment should use image nginx:1.23.3-alpine. Create a new Service resource named cloud-app-svc to expose the cloud-app01 deployment on port 80 in the same namespace. Finally use the following kubectl run command to spin up a utility pod which tests HTTP connectivity through the new service. Confirm that the following command returns an HTTP 200 response code:

kubectl run -n ca2 -i --tty --restart=Never --rm netutil --image cloudacademydevops/networkutils:v2 -- curl -I cloud-app-svc

controlplane $ cat deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cloud-app01
  namespace: ca2
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.23.3-alpine
        ports:
        - containerPort: 80

---

apiVersion: v1
kind: Service
metadata:
  name: cloud-app-svc
  namespace: ca2
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      # By default and for convenience, the `targetPort` is set to
      # the same value as the `port` field.
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane
      # will allocate a port from a range (default: 30000-32767)
      
controlplane $ kubectl run -n ca2 -i --tty --restart=Never --rm netutil --image cloudacademydevops/networkutils:v2 -- curl -I cloud-app-svc
HTTP/1.1 200 OK
Server: nginx/1.23.3
Date: Fri, 16 Aug 2024 14:24:44 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 13 Dec 2022 18:23:05 GMT
Connection: keep-alive
ETag: "6398c309-267"
Accept-Ranges: bytes

pod "netutil" deleted
controlplane $ 



Q9:
Create a StorageClass named class configured with the following settings:

kubernetes.io/aws-ebs provisioner gp2 volume typeretain Reclaim policydo not allow for volume expansion

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: class
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: false


Q10:
Create a PersistentVolume named pv in the qq3 Namespace. The PersistentVolume must be configured with the following settings:

storageClassName: gp21Gi of storage capacityallow a single Node read-write accessuse a hostPath of /mnt/data
The PersistentVolume must be claimed by a PersistentVolumeClaim named pvc. The PersistentVolume must request 1Gi of storage capacity.


apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
  namespace: qq3
spec:
  capacity:
    storage: 21Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: gp
  hostPath:
      path: /mnt/data

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
  namespace: qq3
spec:
  storageClassName: gp
  volumeName: pv
  resources:
    requests:
      storage: 1Gi


Q11:

Create a Pod named pod with one nginx container in the qq2 namespace. The Pod must use the podpvc PersistentVolumeClaim to mount a volume at /data. The pod must have read-only access to the pvc. 

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: qq2
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - name: podpvc
      mountPath: /data
  volumes:
    - name: podpvc
      persistentVolumeClaim:
        claimName: pvc



Q12:
Deploy a pod named nginx using the nginx image in the qq3 namespace with a liveness and readiness probe for port 80.

Pod: nginx Namespace: qq3Image: nginxLiveness Probe: port: 80Readiness Probe: port: 80

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: qq3
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 10
      periodSeconds: 5
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5

Q13:

Create a deployment names nginx with 3 replicas. Get a backup using the etcd. Delete the deployment and restore using your etcd backup 

apt install etcd-client

kubectl create deployment nginx --image=nginx --replicas=3

mkdir etcd-backup

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
                      --cacert=/etc/kubernetes/pki/etcd/ca.crt \
                      --cert=/etc/kubernetes/pki/etcd/server.crt \
                      --key=/etc/kubernetes/pki/etcd/server.key \
snapshot save ./etcd-backup/etcdbackup.db

ETCDCTL_API=3 etcdctl --write-out=table snapshot status ./etcd-backup/etcdbackup.db

kubectl delete deploy nginx

ETCDCTL_API=3 etcdctl snapshot restore etcd-backup/etcdbackup.db --skip-hash-check=true

mkdir temp_yaml_files

mv /etc/kubernetes/manifests/* temp_yaml_files/

cd default.etcd/

mv /var/lib/etcd/member/ /var/lib/etcd/member.bak

mv  member/ /var/lib/etcd/

systemctl stop kubelet

mv temp_yaml_files/* /etc/kubernetes/manifests/

systemctl start kubelet

Q14:

Create a deployment names nginx-deployment in the qq3 namespace. Perform a rolling update to the deployment by setting the image to nginx:alpine.


apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: qq3
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

kubectl set image deployment/nginx-deployment nginx=nginx:alpine

Q15:
Create a new deployment name apache-deployment with an image of caleed httpd:latest

 in the qq3 namespace. Perform rolling update to the image httpd:bookworm and then a rollback of the deployment to restore the original image.


apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache-deployment
  namespace: qq3
  labels:
    app: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: httpd:latest
        ports:
        - containerPort: 80

kubectl set image deployment/apache-deployment httpd=httpd:bookworm -n qq3

then, use the command

kubectl rollout undo deployment/apache-deployment -n qq3


Q16:
Deploy  postgres by using Deployment with the following image:

https://hub.docker.com/_/postgres

Use proper ConfigMap, Secret, Persistent Volume/StorageClass, PersistenVolumeClaim ,  Service



Q17:

Then use elastio/pgadmin to create a database and change for the previous question

https://hub.docker.com/r/elestio/pgadmin


