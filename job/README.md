# kubernetes

# Deploy mysql pod 
kubectl create -f mysql.yaml


# Deploy mysql dump job to create database and insert some data in mysql pod
kubectl create -f mysql-job-dataseed.yaml