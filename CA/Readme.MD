
Cluster Autoscaler vs. other types of Autoscalers
---------------------------------------------------
Before we explore the specifics of CA, let’s review the different types of autoscaling in Kubernetes. They are:

Cluster Autoscaler(CA): adjusts the number of nodes in the cluster when pods fail to schedule or when nodes are underutilized.

Horizontal pod autoscaler (HPA): adjusts the number of replicas of an application.

Vertical pod autoscaler (VPA): adjusts the resource requests and limits of a container.

A simple way to think about the Kubernetes autoscaling functionality is that HPA and VPA operate at the pod level, whereas CA works at the cluster level.

What is the Cluster Autoscaler?
-------------------------------
The cluster autoscaler is a Kubernetes tool that increases or decreases the size of a Kubernetes cluster (by adding or removing nodes), based on the presence of pending pods and node utilization metrics.

The cluster autocaler

Adds nodes to a cluster whenever it detects pending pods that could not be scheduled due to resource shortages.

Removes nodes from a cluster, whenever the utilization of a node falls below a certain threshold defined by the cluster administrator.

Please find below steps.
-------------------------

Step 1: Create EKS Additional IAM policy

Step 2: Attach policy to EKS Node group

Step 3: Add Node group tags

Step 4: Deploy the Cluster Autoscaler in EKS

Step 5: Testing EKS Cluster Autoscaler

NOTE: By using our terraform scripts for clusters creation , you no need to create above first 3 steps we can goto 4th step direct.

Step 1: Create EKS Additional IAM policy
-------------------------------------------
The Cluster Autoscaler requires the following IAM permissions to make calls to AWS APIs on your behalf. Create IAM policy json file:

cat >aws-asg-eks-iam-policy.json<<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:DescribeTags",
                "autoscaling:SetDesiredCapacity",
                "autoscaling:TerminateInstanceInAutoScalingGroup",
                "ec2:DescribeLaunchTemplateVersions"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
EOF

Apply the policy: if you have aws cli and proper permissions

aws iam create-policy --policy-name EKS-Node-group-IAM-policy --policy-document file://aws-asg-eks-iam-policy.json


Step 2: Attach policy to EKS Node group


Step 3: Add Node group tags
----------------------------
The Cluster Autoscaler requires the following tags on your node Auto Scaling groups so that they can be auto-discovered.

Navigate to EKS > Clusters > Clustername > Compute

Select Node Group and click Edit

k8s.io/cluster-autoscaler/<cluster-name>  owned
k8s.io/cluster-autoscaler/enabled         true

Add the Tags at the bottom and save the changes when done.


Step 4: Deploy the Cluster Autoscaler in EKS
----------------------------------------------
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

kubectl get pods -n kube-system -w

You can as well download the yaml file before applying.
------------------------------------------------------
wget https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

kubectl apply -f ./cluster-autoscaler-autodiscover.yaml

Run the following command to add the cluster-autoscaler.kubernetes.io/safe-to-evict annotation to the deployment:
-----------------------------------------------------------------------------------------------------------------
kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false"

Edit the Cluster Autoscaler deployment:
---------------------------------------
kubectl edit deploy cluster-autoscaler -n kube-system

To set your Cluster name instead of <YOUR CLUSTER NAME> in deployment file and add the following options

        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false

Like Below.
----------
    spec:
      containers:
      - command:
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<YOUR CLUSTER NAME>
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false
 

Check to see if the Cluster Autoscaler Pod is running:
-----------------------------------------------------
kubectl get pods -n kube-system -w

Set ClusterAutoscaler version and EKS version:
----------------------------------------------
kubectl -n kube-system set image deployment.apps/cluster-autoscaler cluster-autoscaler=eu.gcr.io/k8s-artifacts-prod/autoscaling/cluster-autoscaler:v1.20.0

Check to see if the Cluster Autoscaler Pod is running:
------------------------------------------------------
kubectl get pods -n kube-system -w

You can watch for logs stream.
-----------------------------
kubectl -n kube-system logs -f deployment.apps/cluster-autoscaler

Step 5: Testing EKS Cluster Autoscaler
--------------------------------------
Now that the installation is complete. Let’s put it to test.

I have two nodes in the cluster. Maximum number set in Node Group is 3.

We’ll deploy a huge number of Pods to see if it will autoscale to maximum number of nodes set in Node group.


cat >nginx-example-autoscale.yaml<<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-to-scaleout
  labels:
    app: nginx
spec:
  replicas: 10
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
EOF

Apply the file and get deployments and pods:
-------------------------------------------
kubectl apply -f nginx-example-autoscale.yaml  
kubectl get deployment/nginx-to-scaleout
kubectl get pods -w

Scale the pods of n number of replicas:
---------------------------------------
kubectl scale --replicas=100 deployment/nginx-to-scaleout

Watch as new pods are running/pending.
-------------------------------------
kubectl get pods -l app=nginx -o wide --watch

You should see a new nodes created and added to the cluster.
------------------------------------------------------------
kubectl get nodes -w

Delete deployment with Pods and the cluster should scale down.
-------------------------------------------------------------
kubectl delete -f nginx-example-autoscale.yaml

You should see a nodes will scaledown and removed from the cluster.
------------------------------------------------------------------
kubectl get nodes -w

Nodes will scaledown automatically but it takes approx 10minutes…

That’s all you need to configure Cluster Autoscaling in an EKS Kubernetes Cluster.