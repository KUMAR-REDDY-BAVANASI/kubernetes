HPA
====
prereqsite: metrics-server
----------------------------
https://github.com/kubernetes-sigs/metrics-server

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl top nodes

kubectl -n kube-system top pods 

cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources:
          limits: 
            cpu: "100m"
          requests:
            cpu: "100m"
EOF

kubectl get pods

kubectl top pods --containers

kubectl expose deploy nginx --port 80 --type NodePort

kubectl get all

cat <<EOF | kubectl apply -f -
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx
spec:
  maxReplicas: 5
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  targetCPUUtilizationPercentage: 20
EOF

[OR]

kubectl autoscale deploy nginx --min 1 --max 5 --cpu-percent 20

kubectl get all 

kubectl describe hpa nginx  

Open separate terminal and generate traffic of nginx website:
-------------------------------------------------------------
siege -q -c 5 -t 2m http://<NODE_EXTERNAL_IP>:<NODE_PORT>

For Example:
------------
siege -q -c 5 -t 2m http://44.203.163.244:30892/

come to previous terminal
-------------------------
kubectl get hpa nginx

kubectl get pods 

kubectl top pods



Cleanup:
--------
kubectl delete hpa nginx

kubectl delete svc nginx

kubectl delete deploy nginx

kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml