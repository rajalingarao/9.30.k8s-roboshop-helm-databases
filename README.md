* Create a roboshop namespace and move to it using kubens.
```
kubectl apply -f namespace.yaml
```
```
kubectl apply -f sc.yaml
```

```
kubens roboshop
```

* EBS Dynamic Provisioning  Steps:
1. We need to install the EBS CSI drivers in EKS cluster.
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.32"
2. Your nodes should have access to connect with EBS volumes. Attach EBS CSI policy to the EC2 instance role.
3. someone on behalf of you should create EBC Volume in AWS and equivalent PV in K8s automatically --> dynamic provisioning that one is Storage class.

$kubectl get sc
$kubectl api-resources

* Important Note: Please add this policy ‘ElasticLoadBalancingFullAccess’ to EKS Worker node's IAM role and execute it.

* Ingress Controller - AWS Load Balancer Controller installation
https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.8/deploy/installation/

1. Create an IAM OIDC provider. You can skip this step if you already have one for your cluster.
```
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster expense \
    --approve
```

2. Download an IAM policy for the LBC using one of the following commands:
If your cluster is in any other region:
```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.10.0/docs/install/iam_policy.json
```

3. Create an IAM policy named AWSLoadBalancerControllerIAMPolicy. If you downloaded a different policy, replace iam-policy with the name of the policy that you downloaded.
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```
* (Optional) Use Existing Policy:
```
aws iam list-policies --query "Policies[?PolicyName=='AWSLoadBalancerControllerIAMPolicy'].Arn" --output text

```
4. Create an IAM role and Kubernetes ServiceAccount for the LBC. Use the ARN from the previous step.
```
eksctl create iamserviceaccount \
--cluster=expense \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::484907532817:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve
```

```
aws eks update-kubeconfig --region us-east-1 --name expense
```

* Helm already installed on k8s workstation.

5. Add the EKS chart repo to Helm
```
helm repo add eks https://aws.github.io/eks-charts
```

6. Helm install command for clusters with IRSA:
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=expense --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```

* check aws-load-balancer-controller is running in kube-system namespace.
```
kubectl get pods -n kube-system
```

* Helm install loop with 60-second delay:
```
for svc in mysql mongodb rabbitmq redis catalogue cart user shipping payment web debug dispatch; do helm install "$svc" "./$svc"; echo "================" ; done
```

* Helm uninstall loop with 60-second delay:
```
for svc in debug dispatch web catalogue cart user shipping payment mysql mongodb rabbitmq redis; do helm uninstall "$svc"; echo ""==============; done
```

* Create a Route 53 record for load balancer:
```
Record name: roboshop.lithesh.shop
Record type: A
Value: dualstack.k8s-roboshop-88b56e4454-917414125.us-east-1.elb.amazonaws.com.
Alias: Yes
TTL (seconds):1
Routing policy: Simple
```
* Run the application in Kubernetes
```
kubens roboshop
```
```
helm install mysql ./mysql
helm install mongodb ./mongodb
helm install rabbitmq ./rabbitmq
helm install redis ./redis
helm install catalogue ./catalogue
helm install cart ./cart
helm install user ./user
helm install shipping ./shipping
helm install payment ./payment
helm install web ./web
helm install debug ./debug
helm install dispatch ./dispatch
```

```
kubectl get pods 
```
```
helm uninstall web ./web
helm uninstall debug ./debug
helm uninstall dispatch ./dispatch
helm uninstall catalogue ./catalogue
helm uninstall cart ./cart
helm uninstall user ./user
helm uninstall shipping ./shipping
helm uninstall payment ./payment
helm uninstall mysql ./mysql
helm uninstall mongodb ./mongodb
helm uninstall rabbitmq ./rabbitmq
helm uninstall redis ./redis
```
