
VPA:
====
prereqsite: metrics-server
----------------------------
https://github.com/kubernetes-sigs/metrics-server

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl top nodes

kubectl -n kube-system top pods 


git clone https://github.com/kubernetes/autoscaler

cd autoscaler/vertical-pod-autoscaler/hack

./vpa-up.sh

kubectl get pods -n kube-system -w

cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-auto-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-auto-deployment
  template:
    metadata:
      labels:
        app: my-auto-deployment
    spec:
      containers:
      - name: my-container
        image: k8s.gcr.io/ubuntu-slim:0.1
        resources:
          requests:
            cpu: 100m
            memory: 50Mi
        command: ["/bin/sh"]
        args: ["-c", "while true; do timeout 0.5s yes >/dev/null; sleep 0.5s; done"]
EOF

kubectl get deploy

kubectl describe pod my-auto-deployment

cat <<EOF | kubectl apply -f -
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       my-auto-deployment
  updatePolicy:
    updateMode: "Auto"
EOF

kubectl get vpa

kubectl decsribe vpa

kubectl describe pod my-auto-deployment

kubectl get pods -w

kubectl get all

kubectl describe pod my-auto-deployment


Cleanup:
=======
kubectl delete vpa my-vpa

kubectl delete deployment my-auto-deployment

./vpa-down.sh

kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml