# My-PV-PVC-Configmaps-Secrets

alias k=kubectl

vi pv.yml
---------   
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  labels:
    type: local
spec:
  storageClassName: standard
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
  persistentVolumeReclaimPolicy: Retain
  	
---> k apply -f pv.yml

--> k get pv

vi pvc.yml
----------
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    type: local
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi

---> k apply -f pvc.yml

vi configmaps.yml
-----------------
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-configmap
data:
  username: sarath
  database: mysqldb

---> k apply -f configmaps.yml

echo -n "Sarath#123" | base64


vi secrets.yml
--------------
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secrets
type: Opaque
data: 
  password: U2FyYXRoIzEyMw==

---> k apply -f secrets.yml

vi deploy.yml
-------------
---
apiVersion: apps/v1
kind: Deployment  
metadata:
  name: mysql  
  labels:         
    app: mysql
spec:
  selector:
    matchLabels:        
      app: mysql
  strategy:
    type: Recreate
  template:       
    metadata:
      labels:   
        app: mysql
    spec:                         
      containers:
      - image: mysql:5.6      
        name: mysql
        env:                     
        - name: MYSQL_ROOT_PASSWORD 
          valueFrom:              
            secretKeyRef:
              name: mysql-secrets
              key: password
        - name: MYSQL_DATABASE
          valueFrom:
            configMapKeyRef:
              name: mysql-configmap
              key: database
        - name: MYSQL_USER
          valueFrom:
            configMapKeyRef:
              name: mysql-configmap
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secrets
              key: password
        ports:
        - containerPort: 3306          
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage 
          mountPath: /var/lib/mysql
      volumes:                     
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
   
---> k apply -f deploy.yml


mysql -u root -p

Sarath#123





(incase if our pod destroys then because of mentioning this strategy in the deployment.yml file another new pod will be creating)
  strategy:
    type: Recreate
							
