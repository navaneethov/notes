## EC2

### Change instance type
- t2.micro to t2.small
- right click on the instance and "change instance type"

### Placement group
- place instances in an AZ
- 3 type
    - cluster -> all instance in that group will be in same cluster, so good n/w throughput, low latency
    - spread -> opposite of cluster, instances will be spread across, good for high availability
    - partition -> 
- edit instance, in advance find an option to select the placement group

### SSH errors

### EC2 instance connect endpoint

### EC2 cloud watch metrics
- CPU, credit, utilization
- Network, in / out
- Status
    - instance status, is up?
    - system status, underlying h/w
    - EBS staus
- Disk, r/w, ops/bytes
    - only for "instance store" ephemeral



### iam, ssm, sns
### example question

alias/aws/ebs

### System Manager - SSM



