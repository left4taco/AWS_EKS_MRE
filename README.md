# AWS_EKS_MRE
This minimal reproducible example of the bug in AWS EKS. 

Steps to reproduce:

1. Create an EKS cluster in AWS. 
   [Link](https://us-west-2.console.aws.amazon.com/eks/home?region=us-west-2#/cluster-create)
   1. Be sure to use the recommended role. 
   2. Select the default VPC and security group.
   3. Enable `Private access` and `Public access`
   4. Assume the name of EKS cluster is **gitlab-runner**

2.  Config `kubelet` command to be able to see the cluster

3. Go to Cloudformation, creata stack-1
   1. Upload `AWS-EKS-CloudFormation.yaml` and click next
   2. Enter stack name `stack-1`. **!important**
   3. Change the name of the cluster to **gitlab-runner** or whatever you set before
   4. Select the default VPC and security group 
   5. Select one or more subnet
   6. Click next and confirm

4. Apply `aws-auth-cm.yaml`
   ```shell
   $ kubelet apply -f aws-auth-cm.yaml
   ```
   Wait for several minutes until you can see the node
   ```shell
   $ kubelet get nodes
   NAME                                         STATUS   ROLES    AGE    VERSION
   ip-172-31-xx-xx.us-west-2.compute.internal   Ready    <none>   9m7s   v1.14.7-eks-1861c5
   ```

5. Create pod in the stack-1
   ```shell
   kubelet apply -f pod/nicolaka_netshoot-ds-1.yaml
   ```
   You can ping anything in the pod
   ```shell
   $ kubectl exec -it nicolaka-netshoot-ds-1-xxx -- bash
   bash# ping www.google.com
   PING www.google.com (172.217.6.68): 56 data bytes
   64 bytes from 172.217.6.68: icmp_seq=0 ttl=56 time=6.847 ms
   ```

6. Create `stack-2` using the same commands in step 3.

   Wait for the node to be ready. 

7. Create the same pod in the stack-2
   ```shell
   kubelet apply -f pod/nicolaka_netshoot-ds-2.yaml
   ```

   Now you can see that the network isn't working in pod2
   ```shell
   $ kubectl exec -it nicolaka-netshoot-ds-2-xxx -- bash
   bash# ping www.google.com
   PING www.google.com (172.217.6.68): 56 data bytes
   ....just wait there
   ```