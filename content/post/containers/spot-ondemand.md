+++
title = 'Critical Workloads on ON-DEMAND Nodes'
date = 2024-10-26T10:00:02Z
draft = false
description = "Critical Workloads on ON_DEMAND Nodes."
featured = false
tags = [
    "kubernetes"
]
categories = ["containers"]
thumbnail = "images/kubernetes.png"
+++
### Optimizing Application Scalability with AWS Spot Instances
If you’re looking to scale your application nodes while keeping costs in check, AWS Spot Instances might be your best bet. In this guide, we’ll explore how to leverage these instances effectively.
<!-- more -->

#### Understanding AWS Instance Types

AWS primarily offers two instance types: **On-Demand** and **Spot**.

- **On-Demand Instances**: These are the most expensive because you pay for them as you use them, providing immediate capacity when needed.
  
- **Spot Instances**: These are unused AWS capacity available at significantly lower prices—typically 80–90% cheaper than On-Demand instances. However, AWS can reclaim these instances when demand increases, usually with about two minutes' notice.

#### The Value of Node Groups

Integrating Spot Instances into your workload can lead to substantial cost savings. Here’s how to structure your node groups effectively:

1. **Critical Workloads**: Always run critical applications on On-Demand instances to ensure reliability. This can be achieved with `affinity`.
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
          - key: eks.amazonaws.com/capacityType
            operator: In
            values:
            - on-demand
``` 
   
2. **Stateful Sets**: These should also be deployed on On-Demand instances.

3. **Diverse Spot Node Groups**: Create multiple node groups for Spot Instances to maximize your chances of availability.

4. **Use Cluster Autoscaler**: Implement the Cluster Autoscaler (CA) to facilitate automatic scaling of your nodes.

5. **Self-Managed Node Groups**: If you're using self-managed node groups, you need to explicitly add the necessary labels. For On-Demand instances, include the following in your Cloud-Init configuration:

```bash
locals {
  labelling_script = <<-EOF
    #!/bin/bash
    export KUBECONFIG=/root/.kube/config
    REGION=us-east-1
    sleep 60
    INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
    NODE_NAME=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID --region $REGION --query 'Reservations[0].Instances[0].PrivateDnsName' --output text)
    NODE_TYPE=$(curl -s --fail http://169.254.169.254/latest/meta-data/instance-life-cycle) 
    sleep 60
    /usr/local/bin/kubectl label nodes $NODE_NAME eks.amazonaws.com/capacityType=$NODE_TYPE
  EOF
}
```

This step is crucial for the scheduler to recognize your instances correctly.

#### Implementing a Real-World Architecture

Let’s consider a practical scenario: you have critical applications running in an On-Demand node group, while less critical cron jobs or monitoring tools operate in a Spot node group.

**Proposed Architecture**: If your application requires 8 replicas, aim for a distribution of 4-5 replicas in the On-Demand node group and 3-4 in the Spot node group. This ensures that if Spot instances are reclaimed, On-Demand instances can manage the load until new Spot instances are available, allowing for “Zero Downtime.”

#### Step-by-Step Implementation

1. **Create Node Groups**: Set up both On-Demand and Spot node groups in your AWS environment.

2. **Check Node Labels**: After creating the node groups, you can verify the labels of a Spot node with this command:

   ```bash
   kubectl describe no ip-100-45-51-226 | grep SPOT 
   ```

   You should see:
   ```
   eks.amazonaws.com/capacityType=SPOT
   ```

   For On-Demand nodes, it will display:
   ```
   eks.amazonaws.com/capacityType=ON_DEMAND
   ```

3. **Deploy Your Application**: Create a sample Nginx deployment with configurations that request the scheduler to allocate approximately 40% of the pods to Spot instances and 60% to On-Demand instances.

   Adjust these percentages based on your specific needs.

4. **Monitor Pod Scheduling**: After deploying your configuration, monitor how the pods are allocated. Ideally, you should see around 6 pods on On-Demand nodes and 4 pods on Spot nodes, as planned.

**Note**: The scheduler allocates pods based on a best-effort basis, so the exact ratio may fluctuate. However, you should achieve a distribution close to your target.

#### Conclusion

Using AWS Spot Instances effectively can lead to significant cost savings while ensuring your applications remain responsive and available. By structuring your node groups thoughtfully and monitoring pod allocations, you can optimize your cloud resources.

