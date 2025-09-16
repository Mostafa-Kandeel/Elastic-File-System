
# Amazon Elastic File System (EFS) README

## Overview
Amazon Elastic File System (EFS) is a scalable, fully managed, cloud-based file storage service provided by AWS. It offers a simple, serverless file system that can be mounted on multiple EC2 instances, Lambda functions, or on-premises servers, supporting NFSv4 protocol. EFS is ideal for shared storage, content management, web serving, or data analytics.

### Key Features
- **Scalability**: Automatically scales storage capacity up or down based on demand.
- **Shared Access**: Supports concurrent access from multiple instances across Availability Zones.
- **Performance Modes**:
  - General Purpose: For latency-sensitive applications (e.g., web servers).
  - Max I/O: For high-throughput workloads (e.g., big data analytics).
- **Storage Classes**:
  - Standard: For frequently accessed data.
  - Infrequent Access (IA): For cost-optimized, less-accessed data.
- **Encryption**: Supports encryption at rest and in transit.
- **Backup**: Integrates with AWS Backup for automated backups.

## Prerequisites
- AWS account with permissions for EFS, EC2, and VPC management.
- EC2 instances or other resources in the same region as the EFS file system.
- Basic knowledge of Linux commands and NFS (Network File System).

## Setup Instructions

### 1. Create an EFS File System
1. **Log in to AWS Management Console**:
   - Navigate to **EFS** > **Create file system**.
2. **Configure File System**:
   - **Name**: Assign a name (e.g., `my-efs`).
   - **VPC**: Select your VPC.
   - **Availability Zones**: Choose subnets (recommended: all AZs for high availability).
   - **Storage Class**: Select Standard or Infrequent Access.
   - **Performance Mode**: Choose General Purpose or Max I/O based on your workload.
   - **Encryption**: Enable encryption at rest (optional, uses AWS KMS).
3. **Tags**: Add tags (e.g., `Environment: Production`).
4. **Review and Create**: Click **Create** to provision the file system.

### 2. Configure Security Groups
1. **Create or Update Security Group**:
   - Go to **EC2** > **Security Groups** > **Create security group**.
   - Add an inbound rule for NFS:
     - **Type**: NFS (port 2049).
     - **Protocol**: TCP.
     - **Source**: Security group of your EC2 instances or a specific CIDR block.
2. **Assign to EFS**:
   - In the EFS console, edit the file system’s **Network** settings to associate the security group.

### 3. Mount EFS on EC2 Instances
1. **Install NFS Client**:
   - On your EC2 instance (e.g., Amazon Linux 2), install the NFS client:
     ```bash
     sudo yum install -y nfs-utils
     ```
     - For Ubuntu:
       ```bash
       sudo apt-get update && sudo apt-get install -y nfs-common
       ```
2. **Create a Mount Point**:
   - Create a directory to mount EFS:
     ```bash
     sudo mkdir /mnt/efs
     ```
3. **Mount the File System**:
   - Get the EFS file system ID or DNS name from the EFS console (e.g., `fs-12345678.efs.us-east-1.amazonaws.com`).
   - Mount EFS:
     ```bash
     sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 fs-12345678.efs.us-east-1.amazonaws.com:/ /mnt/efs
     ```
   - **Explanation**: Uses NFSv4.1 with optimized read/write sizes and retry settings.
4. **Verify Mount**:
   - Check the mount:
     ```bash
     df -h
     ```
   - You should see the EFS file system mounted at `/mnt/efs`.

5. **Persist Mount (Optional)**:
   - Add to `/etc/fstab` for automatic mounting on reboot:
     ```bash
     echo "fs-12345678.efs.us-east-1.amazonaws.com:/ /mnt/efs nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" | sudo tee -a /etc/fstab
     ```
   - Test the `fstab` entry:
     ```bash
     sudo mount -a
     ```

### 4. Enable Automatic Backups (Optional)
1. **Go to AWS Backup**:
   - Navigate to **AWS Backup** > **Create backup plan**.
2. **Configure Plan**:
   - Add a rule to back up the EFS file system (select by resource ID or tag).
   - Set a schedule (e.g., daily) and retention period.
3. **Assign EFS**:
   - Assign the EFS file system to the backup plan.

### 5. Test and Use
- **Write Test Data**:
  - Create a file on one EC2 instance:
    ```bash
    echo "Hello from EFS" | sudo tee /mnt/efs/test.txt
    ```
  - Verify it’s accessible from another EC2 instance mounted to the same EFS:
    ```bash
    cat /mnt/efs/test.txt
    ```
- **Monitor Performance**:
  - Use **CloudWatch** to monitor metrics like `BurstCreditBalance` or `PermittedThroughput`.
  - Adjust performance mode if needed (e.g., switch to Max I/O for high-throughput needs).

## Usage Examples
- **Web Hosting**: Store static assets (e.g., images, CSS) on EFS for access by multiple web servers.
- **Content Management**: Use EFS as a shared file system for CMS platforms like WordPress.
- **Data Analytics**: Store large datasets for processing by multiple EC2 instances.
- **DevOps**: Share configuration files or scripts across CI/CD pipelines.

## Best Practices
- **Security**:
  - Use IAM policies to restrict EFS access.
  - Enable encryption for sensitive data.
  - Limit security group access to trusted sources.
- **Performance**:
  - Choose General Purpose mode for low-latency workloads.
  - Monitor `BurstCreditBalance` to avoid throttling; consider Provisioned Throughput if needed.
- **Cost Optimization**:
  - Use Infrequent Access (IA) with Lifecycle Policies to move old files to cheaper storage.
  - Delete unused file systems to avoid charges.
- **High Availability**:
  - Mount EFS in multiple Availability Zones for redundancy.
  - Use DNS names for mounts to handle AZ failover.

## Troubleshooting
- **Mount Fails**:
  - Verify security group rules allow NFS (port 2049).
  - Ensure the EC2 instance and EFS are in the same VPC and region.
  - Check DNS resolution for the EFS DNS name.
- **Performance Issues**:
  - Monitor CloudWatch metrics for `BurstCreditBalance`.
  - Switch to Max I/O mode or enable Provisioned Throughput for high workloads.
- **Access Denied**:
  - Check IAM roles and EFS file system policies.
  - Verify file permissions on the mounted directory.

## Resources
- [AWS EFS Documentation](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html)
- [AWS Backup for EFS](https://docs.aws.amazon.com/aws-backup/latest/devguide/efs-backup.html)
- [CloudWatch Metrics for EFS](https://docs.aws.amazon.com/efs/latest/ug/monitoring-cloudwatch.html)

## License
This README is provided under the MIT License. Use it to guide your EFS setup and modify as needed for your project.
