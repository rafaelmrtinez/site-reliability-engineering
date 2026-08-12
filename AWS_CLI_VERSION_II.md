# Amazon Web Services CLI Commands Version II

## Table of Contents

- [Create an EC2 Instance](#create-an-ec2-instance)
- [Enable S3 public access by creating a Resource Policy](#enable-s3-public-access-by-creating-a-resource-policy)
- [S3 Bucket hosted website url format](#s3-bucket-hosted-website-url-format)
- [S3 Bucket url format](#s3-bucket-url-format)
- [Hosting a static website using S3](#hosting-a-static-website-using-s3)
- [Create S3 Lifecycle configuration](#create-s3-lifecycle-configuration)
- [Create S3 bucket](#create-s3-bucket)
- [Recover a versioned S3 object by delete the Delete Marker](#recover-a-versioned-s3-object-by-delete-the-delete-marker)
- [List out bucket versions](#list-out-bucket-versions)
- [Filter results with query](#filter-results-with-query)
- [Get bucket configuration](#get-bucket-configuration)
- [Enable bucket versioning](#enable-bucket-versioning)
- [Create a VPC subnet](#create-a-vpc-subnet)
- [Creating a volume](#creating-a-volume)
- [Creating a security group](#creating-a-security-group)
- [Creating a key-pair](#creating-a-key-pair)
- [Updating security group egress rules](#updating-security-group-egress-rules)
- [IAM policy and user access](#iam-policy-and-user-access)
- [RDS resources](#rds-resources)
- [Create an EC2 Instance with employee-app settings](#create-an-ec2-instance-with-employee-app-settings)
- [Install stress for CloudWatch simulation](#install-stress-for-cloudwatch-simulation)
- [Describe Auto Scaling instances](#describe-auto-scaling-instances)
- [EBS volume formatting and mounting flow (simulated)](#ebs-volume-formatting-and-mounting-flow-simulated)

**Create an EC2 Instance**
```bash
aws ec2 run-instances \
--image-id ami-0bdc7d025135d7b49 \
--instance-type t3.micro \
--key-name employee-app-key \
--security-groups employee-app-sg \
--tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=web-server},{Key=Project,Value=employee-app}]'
```

**Enable S3 public access by creating a Resource Policy**
```bash
aws s3api put-public-access-block --bucket kk-static-9664 --public-access-block-configuration '{
    "BlockPublicAcls": false,
    "IgnorePublicAcls": false,
    "BlockPublicPolicy": false,
    "RestrictPublicBuckets": false
}'

aws s3api put-bucket-policy --bucket kk-static-9664 --policy '{
    "Version": "2012-10-17",
    "Statement": [{
        "Sid": "AllowPublic",
        "Principal": "*",
        "Effect": "Allow",
        "Action": [
                "s3:GetObject"
        ],
        "Resource": [
                "arn:aws:s3:::kk-static-9664/*"
        ]
    }]
}'
```

#### Note: You need to lift the block rule to allow the second command to go through. Otherwise you will be blocked from executing that command and would throw an unauthorized error.

**S3 Bucket hosted website url format**
```bash
http://kk-static-9664.s3-website-us-east-1.amazonaws.com/doesnotexist
```

**S3 Bucket url format**
```bash
https://bucket.s3.us-east-1.amazonaws.com/kk-static-9664/index.html
```

**Hosting a static website using S3**
```bash
aws s3api put-bucket-website --bucket kk-static-9664 --website-configuration '{
    "ErrorDocument": {
        "Key": "404.html"
    },
    "IndexDocument": {
        "Suffix": "index.html"
    }
}'
```

**Create S3 Lifecycle configuration**
```bash
aws s3api put-bucket-lifecycle-configuration \
  --bucket kk-lifecycle-john \
  --lifecycle-configuration '{
    "Rules": [
      {
        "ID": "food-lifecycle",
        "Status": "Enabled",
        "Filter": {
          "Prefix": "food/"
        },
        "Transitions": [
          {
            "Days": 30,
            "StorageClass": "STANDARD_IA"
          },
          {
            "Days": 90,
            "StorageClass": "GLACIER_IR"
          }
        ]
      }
    ]
  }'
```

#### Note: Filter will only have contents IF configuration applies to specific files. 

**Create S3 bucket**
```bash
aws s3api create-bucket --bucket kk-lifecycle-john --region us-east-1
```

**Recover a versioned S3 object by delete the Delete Marker**
```bash
aws s3api delete-object --bucket kk-lab2-605680 --key person1.jpg --version-id GDO8LUKy4wwtiY1oNhsv69oXoIUSFJ4Q
```

**List out bucket versions**
```bash
aws s3api list-object-versions --bucket kk-lab2-605680
```

**Filter results with query**
```bash
aws s3api list-object-versions --bucket kk-lab2-605680 --query 'Versions[?Key==`car1.jpg`]
```

**Get bucket configuration**
```bash
aws s3api get-bucket-configuration --bucket kk-lab2-605680
```

**Enable bucket versioning**
```bash
aws s3api put-bucket-versioning \
  --bucket kk-lab2-605680 \
  --versioning-configuration '{
    "Status": "Enabled"
  }'
```

or 

```bash
aws s3api put-bucket-versioning \
  --bucket kk-lab2-605680 \
  --versioning-configuration Status=Suspended
```

#### Note: This is for short hand command of enabling/disabling bucket

**Create a VPC subnet**
```bash
aws ec2 create-subnet \
    --vpc-id vpc-0493d39b4574a7ace \
    --cidr-block 172.31.96.0/20 \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=devops-subnet}]' \
    --dry-run
```


**Creating a volume**
```bash
aws ec2 create-volume \
--volume-type gp3 \
--size 2 \
--tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=xfusion-volume}]' \
--availability-zone us-east-1a --dry-run
```

**Creating a security group**
```bash
aws ec2 create-security-group --description "<security_group_description>" --group-name <security_group_name> --dry-run
```
```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-04e0c5fe753c519f1 \
  --ip-permissions '[
    {
      "IpProtocol": "tcp",
      "FromPort": 80,
      "ToPort": 80,
      "IpRanges": [
        {
          "CidrIp": "0.0.0.0/0"
        }
      ]
    }
  ]' \
  --dry-run
```

**Creating a key-pair**
```bash
aws ec2 create-key-pair --key-name datacenter-kp --key-type rsa --key-format pem --dry-run
```

**Updating security group egress rules**
```bash
aws ec2 update-security-group-rule-descriptions-egress --group-id sg-08517e0e993e08509 --ip-permissions '[
  {
    "IpProtocol": "tcp",
    "FromPort": 80,
    "ToPort": 3306,
    "IpRanges": [
      {
        "Description": "Allow incoming request",
        "CidrIp": "34.234.65.185/32"
      }
    ]
  }
]'
```

```bash
aws ec2 authorize-security-group-egress --group-id sg-08517e0e993e08509 --tag-specifications 'ResourceType=security-group-rule,Tags=[{Key=Project,Value=employee-app}]' --protocol tcp --port 3306 --cidr
```

**IAM policy and user access**
```bash
aws iam create-policy --policy-name AllowUserCreateRDSPolicy
```

```bash
aws iam attach-user-policy --user-name kk_labs_user_248186 --policy-arn arn:aws:iam::745131131958:policy/AllowUserCreateRDSPolicy
```

**RDS resources**
```bash
aws rds create-db-security-group --db-security-group-name employee-app-rds-sg --db-security-group-description 'SG for Employee DB RDS' --tags Key=Project,Value=employee-app
```

```bash
aws rds create-db-instance --db-name employee-app-rds --db-instance-identifier employee-app-rds --db-instance-class db.t3.micro --engine mysql --master-username admin --master-user-password admin --allocated-storage 20
```

**Create an EC2 Instance with employee-app settings**
```bash
aws ec2 run-instances --image-id ami-0bdc7d025135d7b49 --instance-type t3.micro --key-name employee-app --security-groups employee-ap-sg --tag-specifications 'ResourceType=instance,Tags=[{Key=Project,Value=employee-app}]' --dry-run
```

```bash
aws ec2 authorize-security-group-ingress --group-name employee-ap-sg --tag-specifications 'ResourceType=security-group-rule,Tags=[{Key=Project,Value=employee-app}]' --protocol tcp --port 80 --cidr 0.0.0.0/0 --dry-run
```

```bash
aws ec2 create-key-pair --key-name employee-app --key-type rsa --key-format pem --tag-specifications 'ResourceType=key-pair,Tags=[{Key=Project,Value=employee-app}]' --output text --query "KeyMaterial" > employee-app.pem
```

```bash
aws ec2 create-security-group --description "RDS SG" --group-name employee-rds-sg --tag-specifications 'ResourceType=security-group,Tags=[{Key=Project,Value=employee-app}]' --dry-run
```

**Install stress for CloudWatch simulation**
```bash
sudo yum install stress -y
```

```bash
stress --cpu 4 --io 4 --vm-bytes 1G --hdd 2
```

**Describe Auto Scaling instances**
```bash
aws autoscaling describe-auto-scaling-instances
```

Result:
```json
{
    "AutoScalingInstances": [
        {
            "InstanceId": "i-07a9f25e344826958",
            "InstanceType": "t2.small",
            "AutoScalingGroupName": "cloudwatchlab2",
            "AvailabilityZone": "us-east-1a",
            "LifecycleState": "InService",
            "HealthStatus": "HEALTHY",
            "LaunchTemplate": {
                "LaunchTemplateId": "lt-0b760e89a1a43ad80",
                "LaunchTemplateName": "cloudwatch_template",
                "Version": "1"
            },
            "ProtectedFromScaleIn": false
        }
    ]
}
```

**EBS volume formatting and mounting flow (simulated)**

High-level overview:
1. Identify the attached block device using lsblk to confirm the new EBS volume is visible.
2. Create a filesystem on the volume with mkfs.ext4 so it is ready for Linux storage.
3. Create a mount directory such as /mnt/logs to serve as the target location.
4. Mount the formatted volume to the directory and verify the device is attached correctly.
5. Edit /etc/fstab so the volume is automatically mounted after reboot.
6. Unmount the volume temporarily, then reload the fstab entries with mount -a to confirm persistence.
7. Check the final state to confirm the EBS volume is mounted and the system will restore it on startup.

```sh
ec2-user@ip-10-0-0-12 ~ $ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
xvda     202:0    0   20G  0 disk
├─xvda1  202:1    0  18G  0 part /
├─xvda2  202:2    0   1G  0 part /boot
└─xvda3  202:3    0   1G  0 part [SWAP]
xvdb     202:16   0   5G  0 disk /mnt/app-data
xvdc     202:32   0   5G  0 disk
xvdd     202:48   0  10G  0 disk
xvde     202:64   0   1G  0 disk
xvdf     202:80   0   4G  0 disk

ec2-user@ip-10-0-0-12 ~ $ sudo mkfs.ext4 /dev/xvdf
mke2fs 1.46.5 (30-Dec-2021)

Filesystem too small for a journal
Discarding device blocks: done
Creating filesystem with 1024 4k blocks and 1024 inodes

Allocating group tables: done
Writing inode tables: done
Writing superblocks and filesystem accounting information: done

ec2-user@ip-10-0-0-12 ~ $ sudo mkdir -p /mnt/logs

ec2-user@ip-10-0-0-12 ~ $ sudo mount /dev/xvdf /mnt/logs

ec2-user@ip-10-0-0-12 ~ $ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
xvda     202:0    0   20G  0 disk
├─xvda1  202:1    0  18G  0 part /
├─xvda2  202:2    0   1G  0 part /boot
└─xvda3  202:3    0   1G  0 part [SWAP]
xvdb     202:16   0   5G  0 disk /mnt/app-data
xvdc     202:32   0   5G  0 disk
xvdd     202:48   0  10G  0 disk
xvde     202:64   0   1G  0 disk
xvdf     202:80   0   4G  0 disk /mnt/logs

ec2-user@ip-10-0-0-12 ~ $ sudo vi /etc/fstab

ec2-user@ip-10-0-0-12 ~ $ sudo umount /mnt/logs

ec2-user@ip-10-0-0-12 ~ $ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
xvda     202:0    0   20G  0 disk
├─xvda1  202:1    0  18G  0 part /
├─xvda2  202:2    0   1G  0 part /boot
└─xvda3  202:3    0   1G  0 part [SWAP]
xvdb     202:16   0   5G  0 disk /mnt/app-data
xvdc     202:32   0   5G  0 disk
xvdd     202:48   0  10G  0 disk
xvde     202:64   0   1G  0 disk
xvdf     202:80   0   4G  0 disk

ec2-user@ip-10-0-0-12 ~ $ sudo mount -a

ec2-user@ip-10-0-0-12 ~ $ sudo cat /etc/fstab
LABEL=cloudimg-rootfs   /        ext4   discard,errors=remount-ro       0 1
LABEL=UEFI      /boot/efi       vfat    umask=0077      0 1
/dev/xvdf /mnt/logs ext4 rw 0 0
```
