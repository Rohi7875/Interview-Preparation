# AWS Cloud Interview Questions & Answers
### For 7+ Years Experienced Full-Stack Developers

---

## Table of Contents
1. [AWS Basics & Core Services](#aws-basics--core-services)
2. [EC2 & Compute Services](#ec2--compute-services)
3. [S3 & Storage Services](#s3--storage-services)
4. [RDS & Database Services](#rds--database-services)
5. [Lambda & Serverless](#lambda--serverless)
6. [VPC & Networking](#vpc--networking)
7. [IAM & Security](#iam--security)
8. [Load Balancing & Auto Scaling](#load-balancing--auto-scaling)
9. [CloudFront & CDN](#cloudfront--cdn)
10. [Deployment & DevOps](#deployment--devops)
11. [Monitoring & Logging](#monitoring--logging)
12. [Cost Optimization](#cost-optimization)

---

## AWS Basics & Core Services

### Q1. What is AWS and what are its main services?
**Answer:** Amazon Web Services (AWS) is a cloud computing platform offering 200+ services for computing, storage, databases, analytics, networking, and more.

**Core Services Categories:**
- **Compute:** EC2, Lambda, ECS, EKS
- **Storage:** S3, EBS, EFS, Glacier
- **Database:** RDS, DynamoDB, Aurora, ElastiCache
- **Networking:** VPC, Route 53, CloudFront, API Gateway
- **Security:** IAM, KMS, WAF, Shield
- **Management:** CloudWatch, CloudFormation, Systems Manager

**Real-time Example:**
```javascript
// Typical Full-Stack Application Architecture on AWS

/*
┌─────────────────────────────────────────────────────────────┐
│                     USER REQUEST                             │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────────────────┐
│  Route 53 (DNS)                                              │
│  - Domain routing                                            │
│  - Health checks                                             │
└──────────────────┬───────────────────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────────────────┐
│  CloudFront (CDN)                                            │
│  - Static content caching                                    │
│  - SSL/TLS termination                                       │
└──────────────────┬───────────────────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────────────────┐
│  Application Load Balancer (ALB)                             │
│  - Distribute traffic                                        │
│  - Health checks                                             │
└──────────────────┬───────────────────────────────────────────┘
                   │
       ┌───────────┴───────────┐
       ▼                       ▼
┌─────────────┐         ┌─────────────┐
│   EC2       │         │   EC2       │
│  (Node.js)  │         │  (Node.js)  │
│   Server    │         │   Server    │
└──────┬──────┘         └──────┬──────┘
       │                       │
       └───────────┬───────────┘
                   │
       ┌───────────┴───────────┐
       ▼                       ▼
┌─────────────┐         ┌─────────────┐
│  RDS        │         │  ElastiCache│
│  (MySQL)    │         │  (Redis)    │
└─────────────┘         └─────────────┘
       │
       ▼
┌─────────────┐
│  S3         │
│  (Assets)   │
└─────────────┘
*/

// AWS SDK Configuration
const AWS = require('aws-sdk');

// Configure AWS SDK
AWS.config.update({
    region: 'us-east-1',
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
});

// Example: Multi-service AWS application
class AWSApplication {
    constructor() {
        this.s3 = new AWS.S3();
        this.dynamoDB = new AWS.DynamoDB.DocumentClient();
        this.sns = new AWS.SNS();
        this.sqs = new AWS.SQS();
        this.cloudWatch = new AWS.CloudWatch();
    }
    
    // Store file in S3
    async uploadToS3(file, key) {
        const params = {
            Bucket: process.env.S3_BUCKET_NAME,
            Key: key,
            Body: file,
            ContentType: file.mimetype,
            ACL: 'public-read'
        };
        
        try {
            const result = await this.s3.upload(params).promise();
            return result.Location;
        } catch (error) {
            console.error('S3 upload error:', error);
            throw error;
        }
    }
    
    // Store data in DynamoDB
    async saveToDatabase(tableName, item) {
        const params = {
            TableName: tableName,
            Item: item
        };
        
        try {
            await this.dynamoDB.put(params).promise();
            return { success: true, item };
        } catch (error) {
            console.error('DynamoDB error:', error);
            throw error;
        }
    }
    
    // Send notification via SNS
    async sendNotification(topic, message) {
        const params = {
            TopicArn: topic,
            Message: JSON.stringify(message),
            Subject: 'Application Notification'
        };
        
        try {
            await this.sns.publish(params).promise();
            return { success: true };
        } catch (error) {
            console.error('SNS error:', error);
            throw error;
        }
    }
    
    // Send message to queue
    async sendToQueue(queueUrl, message) {
        const params = {
            QueueUrl: queueUrl,
            MessageBody: JSON.stringify(message)
        };
        
        try {
            await this.sqs.sendMessage(params).promise();
            return { success: true };
        } catch (error) {
            console.error('SQS error:', error);
            throw error;
        }
    }
    
    // Log custom metrics to CloudWatch
    async logMetric(metricName, value) {
        const params = {
            MetricData: [{
                MetricName: metricName,
                Value: value,
                Unit: 'Count',
                Timestamp: new Date()
            }],
            Namespace: 'MyApplication'
        };
        
        try {
            await this.cloudWatch.putMetricData(params).promise();
        } catch (error) {
            console.error('CloudWatch error:', error);
        }
    }
}

// Usage
const app = new AWSApplication();

// Example: User registration flow
async function registerUser(userData, profileImage) {
    try {
        // 1. Upload profile image to S3
        const imageUrl = await app.uploadToS3(
            profileImage,
            `profiles/${userData.id}/avatar.jpg`
        );
        
        // 2. Save user data to DynamoDB
        const user = {
            id: userData.id,
            name: userData.name,
            email: userData.email,
            profileImage: imageUrl,
            createdAt: Date.now()
        };
        
        await app.saveToDatabase('Users', user);
        
        // 3. Send welcome email via SNS
        await app.sendNotification(
            process.env.SNS_TOPIC_ARN,
            {
                type: 'WELCOME_EMAIL',
                email: userData.email,
                name: userData.name
            }
        );
        
        // 4. Queue background tasks
        await app.sendToQueue(
            process.env.SQS_QUEUE_URL,
            {
                type: 'PROCESS_NEW_USER',
                userId: userData.id
            }
        );
        
        // 5. Log metric
        await app.logMetric('UserRegistrations', 1);
        
        return { success: true, user };
    } catch (error) {
        console.error('Registration error:', error);
        throw error;
    }
}
```

---

## EC2 & Compute Services

### Q2. What is EC2 and how do you deploy applications on it?
**Answer:** Amazon EC2 (Elastic Compute Cloud) provides scalable computing capacity. It's like renting virtual servers in the cloud.

**Key Concepts:**
- **Instances:** Virtual servers with various configurations
- **AMI (Amazon Machine Image):** Template for the root volume
- **Instance Types:** Different CPU, memory, storage combinations
- **Security Groups:** Virtual firewalls
- **Key Pairs:** SSH access credentials

**Real-time Example:**
```bash
# 1. Launch EC2 Instance (via AWS CLI)
aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --instance-type t2.micro \
    --key-name MyKeyPair \
    --security-group-ids sg-903004f8 \
    --subnet-id subnet-6e7f829e \
    --user-data file://setup-script.sh

# 2. Setup Script (setup-script.sh)
#!/bin/bash

# Update system
yum update -y

# Install Node.js
curl -sL https://rpm.nodesource.com/setup_18.x | bash -
yum install -y nodejs

# Install PM2 for process management
npm install -g pm2

# Install Git
yum install -y git

# Clone application
cd /home/ec2-user
git clone https://github.com/username/app.git
cd app

# Install dependencies
npm install

# Setup environment variables
cat > .env << EOF
NODE_ENV=production
PORT=3000
DB_HOST=${DB_HOST}
DB_USER=${DB_USER}
DB_PASSWORD=${DB_PASSWORD}
EOF

# Start application with PM2
pm2 start npm --name "myapp" -- start
pm2 save
pm2 startup

# Configure nginx as reverse proxy
yum install -y nginx
cat > /etc/nginx/conf.d/app.conf << EOF
server {
    listen 80;
    server_name _;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_cache_bypass \$http_upgrade;
    }
}
EOF

# Start nginx
systemctl start nginx
systemctl enable nginx
```

```javascript
// Node.js Application Deployment Script
const AWS = require('aws-sdk');
const ec2 = new AWS.EC2({ region: 'us-east-1' });

class EC2Deployer {
    // Create new EC2 instance
    async launchInstance(config) {
        const params = {
            ImageId: config.amiId,
            InstanceType: config.instanceType,
            KeyName: config.keyName,
            MinCount: 1,
            MaxCount: 1,
            SecurityGroupIds: config.securityGroups,
            SubnetId: config.subnetId,
            UserData: Buffer.from(config.userDataScript).toString('base64'),
            TagSpecifications: [{
                ResourceType: 'instance',
                Tags: [
                    { Key: 'Name', Value: config.name },
                    { Key: 'Environment', Value: config.environment },
                    { Key: 'Application', Value: config.appName }
                ]
            }]
        };
        
        try {
            const result = await ec2.runInstances(params).promise();
            const instanceId = result.Instances[0].InstanceId;
            
            console.log(`Instance ${instanceId} launched successfully`);
            
            // Wait for instance to be running
            await this.waitForInstance(instanceId, 'running');
            
            // Get public IP
            const instance = await this.getInstanceDetails(instanceId);
            
            return {
                instanceId,
                publicIp: instance.PublicIpAddress,
                privateIp: instance.PrivateIpAddress
            };
        } catch (error) {
            console.error('Launch instance error:', error);
            throw error;
        }
    }
    
    // Wait for instance state
    async waitForInstance(instanceId, state) {
        const params = {
            InstanceIds: [instanceId]
        };
        
        await ec2.waitFor(`instance${state}`, params).promise();
        console.log(`Instance ${instanceId} is ${state}`);
    }
    
    // Get instance details
    async getInstanceDetails(instanceId) {
        const params = {
            InstanceIds: [instanceId]
        };
        
        const result = await ec2.describeInstances(params).promise();
        return result.Reservations[0].Instances[0];
    }
    
    // Stop instance
    async stopInstance(instanceId) {
        const params = {
            InstanceIds: [instanceId]
        };
        
        await ec2.stopInstances(params).promise();
        console.log(`Instance ${instanceId} stopped`);
    }
    
    // Terminate instance
    async terminateInstance(instanceId) {
        const params = {
            InstanceIds: [instanceId]
        };
        
        await ec2.terminateInstances(params).promise();
        console.log(`Instance ${instanceId} terminated`);
    }
    
    // Create AMI from instance
    async createAMI(instanceId, name) {
        const params = {
            InstanceId: instanceId,
            Name: name,
            Description: `AMI created from ${instanceId}`,
            NoReboot: true
        };
        
        const result = await ec2.createImage(params).promise();
        console.log(`AMI ${result.ImageId} created`);
        
        return result.ImageId;
    }
}

// Usage
const deployer = new EC2Deployer();

const config = {
    amiId: 'ami-0c55b159cbfafe1f0',
    instanceType: 't2.micro',
    keyName: 'MyKeyPair',
    securityGroups: ['sg-903004f8'],
    subnetId: 'subnet-6e7f829e',
    name: 'WebServer01',
    environment: 'production',
    appName: 'MyApp',
    userDataScript: `#!/bin/bash
        yum update -y
        # Install and configure application
    `
};

// Deploy new instance
deployer.launchInstance(config)
    .then(result => {
        console.log('Deployment successful:', result);
    })
    .catch(error => {
        console.error('Deployment failed:', error);
    });
```

---

## S3 & Storage Services

### Q3. How do you use S3 for file storage and static website hosting?
**Answer:** Amazon S3 (Simple Storage Service) is object storage with high durability, availability, and scalability.

**Key Features:**
- 99.999999999% (11 9's) durability
- Unlimited storage
- Object versioning
- Lifecycle policies
- Server-side encryption
- Static website hosting

**Real-time Example:**
```javascript
// Complete S3 File Management System
const AWS = require('aws-sdk');
const multer = require('multer');
const multerS3 = require('multer-s3');
const path = require('path');

const s3 = new AWS.S3({
    region: 'us-east-1',
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
});

class S3FileManager {
    constructor(bucketName) {
        this.bucketName = bucketName;
    }
    
    // Upload file to S3
    async uploadFile(file, folder = '') {
        const key = folder ? `${folder}/${Date.now()}_${file.originalname}` : 
                            `${Date.now()}_${file.originalname}`;
        
        const params = {
            Bucket: this.bucketName,
            Key: key,
            Body: file.buffer,
            ContentType: file.mimetype,
            ACL: 'public-read',
            Metadata: {
                originalName: file.originalname,
                uploadDate: new Date().toISOString()
            }
        };
        
        try {
            const result = await s3.upload(params).promise();
            
            return {
                success: true,
                url: result.Location,
                key: result.Key,
                bucket: result.Bucket,
                etag: result.ETag
            };
        } catch (error) {
            console.error('Upload error:', error);
            throw error;
        }
    }
    
    // Upload multiple files
    async uploadMultipleFiles(files, folder = '') {
        const uploadPromises = files.map(file => 
            this.uploadFile(file, folder)
        );
        
        return await Promise.all(uploadPromises);
    }
    
    // Download file from S3
    async downloadFile(key) {
        const params = {
            Bucket: this.bucketName,
            Key: key
        };
        
        try {
            const result = await s3.getObject(params).promise();
            return result.Body;
        } catch (error) {
            console.error('Download error:', error);
            throw error;
        }
    }
    
    // Get signed URL for private files
    async getSignedUrl(key, expiresIn = 3600) {
        const params = {
            Bucket: this.bucketName,
            Key: key,
            Expires: expiresIn
        };
        
        return s3.getSignedUrl('getObject', params);
    }
    
    // Delete file from S3
    async deleteFile(key) {
        const params = {
            Bucket: this.bucketName,
            Key: key
        };
        
        try {
            await s3.deleteObject(params).promise();
            return { success: true, deleted: key };
        } catch (error) {
            console.error('Delete error:', error);
            throw error;
        }
    }
    
    // Delete multiple files
    async deleteMultipleFiles(keys) {
        const params = {
            Bucket: this.bucketName,
            Delete: {
                Objects: keys.map(key => ({ Key: key }))
            }
        };
        
        try {
            const result = await s3.deleteObjects(params).promise();
            return result.Deleted;
        } catch (error) {
            console.error('Delete multiple error:', error);
            throw error;
        }
    }
    
    // List files in bucket/folder
    async listFiles(prefix = '', maxKeys = 1000) {
        const params = {
            Bucket: this.bucketName,
            Prefix: prefix,
            MaxKeys: maxKeys
        };
        
        try {
            const result = await s3.listObjectsV2(params).promise();
            
            return result.Contents.map(item => ({
                key: item.Key,
                size: item.Size,
                lastModified: item.LastModified,
                etag: item.ETag
            }));
        } catch (error) {
            console.error('List error:', error);
            throw error;
        }
    }
    
    // Copy file within S3
    async copyFile(sourceKey, destinationKey) {
        const params = {
            Bucket: this.bucketName,
            CopySource: `${this.bucketName}/${sourceKey}`,
            Key: destinationKey
        };
        
        try {
            await s3.copyObject(params).promise();
            return { success: true, copied: destinationKey };
        } catch (error) {
            console.error('Copy error:', error);
            throw error;
        }
    }
    
    // Get file metadata
    async getFileMetadata(key) {
        const params = {
            Bucket: this.bucketName,
            Key: key
        };
        
        try {
            const result = await s3.headObject(params).promise();
            
            return {
                contentType: result.ContentType,
                contentLength: result.ContentLength,
                lastModified: result.LastModified,
                etag: result.ETag,
                metadata: result.Metadata
            };
        } catch (error) {
            console.error('Metadata error:', error);
            throw error;
        }
    }
    
    // Create presigned POST for direct browser upload
    async createPresignedPost(key, conditions = {}) {
        const params = {
            Bucket: this.bucketName,
            Fields: {
                key: key
            },
            Conditions: [
                ['content-length-range', 0, 10485760], // 10MB max
                ...Object.entries(conditions)
            ],
            Expires: 3600
        };
        
        return new Promise((resolve, reject) => {
            s3.createPresignedPost(params, (err, data) => {
                if (err) reject(err);
                else resolve(data);
            });
        });
    }
}

// Express.js integration
const express = require('express');
const app = express();

const fileManager = new S3FileManager('my-bucket-name');

// Multer S3 configuration for direct upload
const upload = multer({
    storage: multerS3({
        s3: s3,
        bucket: 'my-bucket-name',
        acl: 'public-read',
        metadata: (req, file, cb) => {
            cb(null, { fieldName: file.fieldname });
        },
        key: (req, file, cb) => {
            const fileName = Date.now() + path.extname(file.originalname);
            cb(null, `uploads/${fileName}`);
        }
    }),
    limits: { fileSize: 10 * 1024 * 1024 }, // 10MB limit
    fileFilter: (req, file, cb) => {
        const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'application/pdf'];
        
        if (allowedTypes.includes(file.mimetype)) {
            cb(null, true);
        } else {
            cb(new Error('Invalid file type'));
        }
    }
});

// Upload single file
app.post('/upload', upload.single('file'), (req, res) => {
    res.json({
        success: true,
        file: {
            url: req.file.location,
            key: req.file.key,
            size: req.file.size
        }
    });
});

// Upload multiple files
app.post('/upload-multiple', upload.array('files', 10), (req, res) => {
    res.json({
        success: true,
        files: req.files.map(file => ({
            url: file.location,
            key: file.key,
            size: file.size
        }))
    });
});

// Get presigned URL for download
app.get('/download/:key', async (req, res) => {
    try {
        const url = await fileManager.getSignedUrl(req.params.key, 300);
        res.json({ url });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// Delete file
app.delete('/files/:key', async (req, res) => {
    try {
        await fileManager.deleteFile(req.params.key);
        res.json({ success: true });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// List files
app.get('/files', async (req, res) => {
    try {
        const files = await fileManager.listFiles(req.query.prefix);
        res.json({ files });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});
```

This is a comprehensive start to the AWS guide. Would you like me to continue with more sections including Lambda, RDS, IAM, CloudWatch, and deployment strategies?
