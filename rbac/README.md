# Example to test new user accessing k8s cluster or not in aws eks cluster
aws iam create-user --user-name rbac-user

aws iam create-access-key --user-name rbac-user | tee /tmp/create_output.json

# output
```
{
    "AccessKey": {
        "UserName": "rbac-user",
        "Status": "Active",
        "CreateDate": "2019-07-17T15:37:27Z",
        "SecretAccessKey": < AWS Secret Access Key > ,
        "AccessKeyId": < AWS Access Key >
    }
}
```

# Export the keys to a file
```
cat << EoF > rbacuser_creds.sh
export AWS_SECRET_ACCESS_KEY=$(jq -r .AccessKey.SecretAccessKey /tmp/create_output.json)
export AWS_ACCESS_KEY_ID=$(jq -r .AccessKey.AccessKeyId /tmp/create_output.json)
EoF
```

# Get the configmap and replace userarn
kubectl get configmap -n kube-system aws-auth -o yaml | grep -v "creationTimestamp\|resourceVersion\|selfLink\|uid" | sed '/^  annotations:/,+2 d' > aws-auth.yaml

```
cat << EoF >> aws-auth.yaml
data:
  mapUsers: |
    - userarn: arn:aws:iam::${ACCOUNT_ID}:user/rbac-user
      username: rbac-user
EoF
```

cat aws-auth.yaml

# deploy aws-auth configuration in eks cluster
kubectl apply -f aws-auth.yaml

./rbacuser_creds.sh

aws sts get-caller-identity

kubectl get pods -n rbac-test


# Example like this
```
apiVersion: v1
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::<-ACCOUNT-No->:role/eksdefaultspotBlueprintsNodeInstanceRole
      username: system:node:{{EC2PrivateDNSName}}
  mapUsers: |
    - userarn: arn:aws:iam::<-ACCOUNT-No->:user/admin
      username: kumar
      groups:
      - kumar-finance
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapUsers: |
    - userarn: arn:aws:iam::<-ACCOUNT-No->:user/rbac-user
      username: rbac-user
```