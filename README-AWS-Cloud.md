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

---

## RDS & Database Services

### Q4. What is Amazon RDS and how do you manage databases?
**Answer:** Amazon RDS (Relational Database Service) is a managed database service supporting MySQL, PostgreSQL, MariaDB, Oracle, and SQL Server.

**Key Features:**
- Automated backups and snapshots
- Multi-AZ deployments for high availability
- Read replicas for read scaling
- Automated software patching
- Encryption at rest and in transit
- Performance Insights

**Real-time Example:**
```javascript
// RDS Database Management System
const AWS = require('aws-sdk');
const mysql = require('mysql2/promise');

const rds = new AWS.RDS({ region: 'us-east-1' });

class RDSManager {
    constructor() {
        this.rds = rds;
    }
    
    // Create RDS instance
    async createDBInstance(config) {
        const params = {
            DBInstanceIdentifier: config.identifier,
            DBInstanceClass: config.instanceClass,
            Engine: config.engine,
            MasterUsername: config.username,
            MasterUserPassword: config.password,
            AllocatedStorage: config.storage,
            VpcSecurityGroupIds: config.securityGroups,
            DBSubnetGroupName: config.subnetGroup,
            BackupRetentionPeriod: config.backupRetention,
            MultiAZ: config.multiAZ,
            StorageEncrypted: true,
            Tags: [
                { Key: 'Environment', Value: config.environment },
                { Key: 'Application', Value: config.application }
            ]
        };
        
        try {
            const result = await this.rds.createDBInstance(params).promise();
            console.log(`RDS instance ${config.identifier} created`);
            return result.DBInstance;
        } catch (error) {
            console.error('RDS creation error:', error);
            throw error;
        }
    }
    
    // Create read replica
    async createReadReplica(sourceInstanceId, replicaId) {
        const params = {
            DBInstanceIdentifier: replicaId,
            SourceDBInstanceIdentifier: sourceInstanceId,
            DBInstanceClass: 'db.t3.micro'
        };
        
        try {
            const result = await this.rds.createDBInstanceReadReplica(params).promise();
            console.log(`Read replica ${replicaId} created`);
            return result.DBInstance;
        } catch (error) {
            console.error('Read replica creation error:', error);
            throw error;
        }
    }
    
    // Create database connection
    async createConnection(config) {
        const connection = await mysql.createConnection({
            host: config.endpoint,
            user: config.username,
            password: config.password,
            database: config.database,
            ssl: { rejectUnauthorized: false }
        });
        
        return connection;
    }
    
    // Execute query
    async executeQuery(connection, query, params = []) {
        try {
            const [rows] = await connection.execute(query, params);
            return rows;
        } catch (error) {
            console.error('Query execution error:', error);
            throw error;
        }
    }
    
    // Create snapshot
    async createSnapshot(instanceId, snapshotId) {
        const params = {
            DBInstanceIdentifier: instanceId,
            DBSnapshotIdentifier: snapshotId
        };
        
        try {
            const result = await this.rds.createDBSnapshot(params).promise();
            console.log(`Snapshot ${snapshotId} created`);
            return result.DBSnapshot;
        } catch (error) {
            console.error('Snapshot creation error:', error);
            throw error;
        }
    }
    
    // Restore from snapshot
    async restoreFromSnapshot(snapshotId, newInstanceId) {
        const params = {
            DBInstanceIdentifier: newInstanceId,
            DBSnapshotIdentifier: snapshotId,
            DBInstanceClass: 'db.t3.micro'
        };
        
        try {
            const result = await this.rds.restoreDBInstanceFromDBSnapshot(params).promise();
            console.log(`Instance restored from snapshot ${snapshotId}`);
            return result.DBInstance;
        } catch (error) {
            console.error('Restore error:', error);
            throw error;
        }
    }
}

// Usage example
const rdsManager = new RDSManager();

// Create RDS instance
const dbConfig = {
    identifier: 'myapp-db',
    instanceClass: 'db.t3.micro',
    engine: 'mysql',
    username: 'admin',
    password: 'securepassword123',
    storage: 20,
    securityGroups: ['sg-12345678'],
    subnetGroup: 'default',
    backupRetention: 7,
    multiAZ: false,
    environment: 'production',
    application: 'MyApp'
};

// Database operations
class DatabaseService {
    constructor(connection) {
        this.connection = connection;
    }
    
    // User management
    async createUser(userData) {
        const query = `
            INSERT INTO users (name, email, created_at) 
            VALUES (?, ?, NOW())
        `;
        
        const result = await this.connection.execute(query, [
            userData.name, 
            userData.email
        ]);
        
        return result[0].insertId;
    }
    
    async getUserById(id) {
        const query = 'SELECT * FROM users WHERE id = ?';
        const [rows] = await this.connection.execute(query, [id]);
        return rows[0];
    }
    
    async updateUser(id, userData) {
        const query = `
            UPDATE users 
            SET name = ?, email = ?, updated_at = NOW() 
            WHERE id = ?
        `;
        
        await this.connection.execute(query, [
            userData.name, 
            userData.email, 
            id
        ]);
    }
    
    async deleteUser(id) {
        const query = 'DELETE FROM users WHERE id = ?';
        await this.connection.execute(query, [id]);
    }
    
    // Transaction example
    async transferMoney(fromUserId, toUserId, amount) {
        await this.connection.beginTransaction();
        
        try {
            // Deduct from sender
            await this.connection.execute(
                'UPDATE accounts SET balance = balance - ? WHERE user_id = ?',
                [amount, fromUserId]
            );
            
            // Add to receiver
            await this.connection.execute(
                'UPDATE accounts SET balance = balance + ? WHERE user_id = ?',
                [amount, toUserId]
            );
            
            // Log transaction
            await this.connection.execute(
                'INSERT INTO transactions (from_user, to_user, amount, created_at) VALUES (?, ?, ?, NOW())',
                [fromUserId, toUserId, amount]
            );
            
            await this.connection.commit();
            return { success: true };
        } catch (error) {
            await this.connection.rollback();
            throw error;
        }
    }
}
```

---

## Lambda & Serverless

### Q5. What is AWS Lambda and how do you build serverless applications?
**Answer:** AWS Lambda is a serverless compute service that runs code without provisioning servers. You pay only for compute time consumed.

**Key Features:**
- Event-driven execution
- Automatic scaling
- Pay-per-request pricing
- Multiple runtime support
- Integration with 200+ AWS services

**Real-time Example:**
```javascript
// Lambda Functions for Full-Stack Application

// 1. API Gateway + Lambda Handler
exports.handler = async (event, context) => {
    const { httpMethod, path, body, queryStringParameters } = event;
    
    try {
        switch (httpMethod) {
            case 'GET':
                if (path === '/users') {
                    return await getUsers(queryStringParameters);
                }
                break;
                
            case 'POST':
                if (path === '/users') {
                    return await createUser(JSON.parse(body));
                }
                break;
                
            case 'PUT':
                if (path.startsWith('/users/')) {
                    const userId = path.split('/')[2];
                    return await updateUser(userId, JSON.parse(body));
                }
                break;
                
            case 'DELETE':
                if (path.startsWith('/users/')) {
                    const userId = path.split('/')[2];
                    return await deleteUser(userId);
                }
                break;
        }
        
        return {
            statusCode: 404,
            body: JSON.stringify({ error: 'Not Found' })
        };
    } catch (error) {
        return {
            statusCode: 500,
            body: JSON.stringify({ error: error.message })
        };
    }
};

// 2. S3 Event Trigger Lambda
exports.s3ImageProcessor = async (event) => {
    const AWS = require('aws-sdk');
    const sharp = require('sharp');
    
    const s3 = new AWS.S3();
    
    for (const record of event.Records) {
        const bucket = record.s3.bucket.name;
        const key = record.s3.object.key;
        
        try {
            // Download original image
            const originalImage = await s3.getObject({
                Bucket: bucket,
                Key: key
            }).promise();
            
            // Create thumbnail
            const thumbnail = await sharp(originalImage.Body)
                .resize(200, 200)
                .jpeg({ quality: 80 })
                .toBuffer();
            
            // Upload thumbnail
            await s3.putObject({
                Bucket: bucket,
                Key: `thumbnails/${key}`,
                Body: thumbnail,
                ContentType: 'image/jpeg'
            }).promise();
            
            console.log(`Thumbnail created for ${key}`);
        } catch (error) {
            console.error(`Error processing ${key}:`, error);
        }
    }
};

// 3. DynamoDB Stream Trigger Lambda
exports.dynamoDBStreamHandler = async (event) => {
    const AWS = require('aws-sdk');
    const sns = new AWS.SNS();
    
    for (const record of event.Records) {
        if (record.eventName === 'INSERT') {
            const newUser = record.dynamodb.NewImage;
            
            // Send welcome email
            await sns.publish({
                TopicArn: process.env.WELCOME_EMAIL_TOPIC,
                Message: JSON.stringify({
                    type: 'WELCOME_EMAIL',
                    email: newUser.email.S,
                    name: newUser.name.S
                }),
                Subject: 'Welcome to Our Platform!'
            }).promise();
            
            console.log(`Welcome email queued for ${newUser.email.S}`);
        }
    }
};

// 4. Scheduled Lambda (Cron Job)
exports.scheduledTask = async (event) => {
    const AWS = require('aws-sdk');
    const dynamodb = new AWS.DynamoDB.DocumentClient();
    
    // Clean up old sessions
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - 30);
    
    const params = {
        TableName: 'user_sessions',
        FilterExpression: 'created_at < :cutoff',
        ExpressionAttributeValues: {
            ':cutoff': cutoffDate.toISOString()
        }
    };
    
    const result = await dynamodb.scan(params).promise();
    
    for (const session of result.Items) {
        await dynamodb.delete({
            TableName: 'user_sessions',
            Key: { session_id: session.session_id }
        }).promise();
    }
    
    console.log(`Cleaned up ${result.Items.length} old sessions`);
};

// 5. Lambda with Layers (Shared Dependencies)
const AWS = require('aws-sdk');
const mysql = require('mysql2/promise');

// Database connection pool
let connectionPool = null;

const getConnection = async () => {
    if (!connectionPool) {
        connectionPool = mysql.createPool({
            host: process.env.DB_HOST,
            user: process.env.DB_USER,
            password: process.env.DB_PASSWORD,
            database: process.env.DB_NAME,
            waitForConnections: true,
            connectionLimit: 10,
            queueLimit: 0
        });
    }
    return connectionPool;
};

exports.databaseHandler = async (event, context) => {
    const connection = await getConnection();
    
    try {
        const [rows] = await connection.execute(
            'SELECT * FROM users WHERE active = ?',
            [true]
        );
        
        return {
            statusCode: 200,
            body: JSON.stringify(rows)
        };
    } catch (error) {
        return {
            statusCode: 500,
            body: JSON.stringify({ error: error.message })
        };
    }
};

// 6. Lambda with Step Functions
exports.stepFunctionHandler = async (event) => {
    const { taskType, data } = event;
    
    switch (taskType) {
        case 'PROCESS_ORDER':
            return await processOrder(data);
            
        case 'SEND_NOTIFICATION':
            return await sendNotification(data);
            
        case 'UPDATE_INVENTORY':
            return await updateInventory(data);
            
        default:
            throw new Error(`Unknown task type: ${taskType}`);
    }
};

async function processOrder(orderData) {
    // Process order logic
    return {
        orderId: orderData.orderId,
        status: 'PROCESSED',
        processedAt: new Date().toISOString()
    };
}

async function sendNotification(notificationData) {
    const AWS = require('aws-sdk');
    const sns = new AWS.SNS();
    
    await sns.publish({
        TopicArn: process.env.NOTIFICATION_TOPIC,
        Message: notificationData.message,
        Subject: notificationData.subject
    }).promise();
    
    return { notificationSent: true };
}

async function updateInventory(inventoryData) {
    // Update inventory logic
    return {
        productId: inventoryData.productId,
        quantity: inventoryData.quantity,
        updatedAt: new Date().toISOString()
    };
}
```

---

## VPC & Networking

### Q6. What is VPC and how do you configure networking?
**Answer:** Amazon VPC (Virtual Private Cloud) lets you provision a logically isolated section of AWS where you can launch resources in a virtual network.

**Key Components:**
- **Subnets:** Network segments within VPC
- **Route Tables:** Control traffic routing
- **Internet Gateway:** Internet access for VPC
- **NAT Gateway:** Outbound internet for private subnets
- **Security Groups:** Instance-level firewall
- **NACLs:** Subnet-level firewall

**Real-time Example:**
```javascript
// VPC Configuration and Management
const AWS = require('aws-sdk');
const ec2 = new AWS.EC2({ region: 'us-east-1' });

class VPCManager {
    constructor() {
        this.ec2 = ec2;
    }
    
    // Create VPC
    async createVPC(name, cidrBlock) {
        const params = {
            CidrBlock: cidrBlock,
            TagSpecifications: [{
                ResourceType: 'vpc',
                Tags: [
                    { Key: 'Name', Value: name },
                    { Key: 'Environment', Value: 'production' }
                ]
            }]
        };
        
        try {
            const result = await this.ec2.createVpc(params).promise();
            const vpcId = result.Vpc.VpcId;
            
            // Enable DNS hostnames
            await this.ec2.modifyVpcAttribute({
                VpcId: vpcId,
                EnableDnsHostnames: { Value: true }
            }).promise();
            
            console.log(`VPC ${vpcId} created`);
            return result.Vpc;
        } catch (error) {
            console.error('VPC creation error:', error);
            throw error;
        }
    }
    
    // Create Internet Gateway
    async createInternetGateway(name) {
        const params = {
            TagSpecifications: [{
                ResourceType: 'internet-gateway',
                Tags: [{ Key: 'Name', Value: name }]
            }]
        };
        
        try {
            const result = await this.ec2.createInternetGateway(params).promise();
            console.log(`Internet Gateway ${result.InternetGateway.InternetGatewayId} created`);
            return result.InternetGateway;
        } catch (error) {
            console.error('IGW creation error:', error);
            throw error;
        }
    }
    
    // Attach Internet Gateway to VPC
    async attachInternetGateway(vpcId, igwId) {
        const params = {
            InternetGatewayId: igwId,
            VpcId: vpcId
        };
        
        try {
            await this.ec2.attachInternetGateway(params).promise();
            console.log(`Internet Gateway ${igwId} attached to VPC ${vpcId}`);
        } catch (error) {
            console.error('IGW attachment error:', error);
            throw error;
        }
    }
    
    // Create Subnet
    async createSubnet(vpcId, cidrBlock, availabilityZone, name) {
        const params = {
            VpcId: vpcId,
            CidrBlock: cidrBlock,
            AvailabilityZone: availabilityZone,
            TagSpecifications: [{
                ResourceType: 'subnet',
                Tags: [{ Key: 'Name', Value: name }]
            }]
        };
        
        try {
            const result = await this.ec2.createSubnet(params).promise();
            console.log(`Subnet ${result.Subnet.SubnetId} created`);
            return result.Subnet;
        } catch (error) {
            console.error('Subnet creation error:', error);
            throw error;
        }
    }
    
    // Create Route Table
    async createRouteTable(vpcId, name) {
        const params = {
            VpcId: vpcId,
            TagSpecifications: [{
                ResourceType: 'route-table',
                Tags: [{ Key: 'Name', Value: name }]
            }]
        };
        
        try {
            const result = await this.ec2.createRouteTable(params).promise();
            console.log(`Route Table ${result.RouteTable.RouteTableId} created`);
            return result.RouteTable;
        } catch (error) {
            console.error('Route table creation error:', error);
            throw error;
        }
    }
    
    // Create Route
    async createRoute(routeTableId, destinationCidrBlock, gatewayId) {
        const params = {
            RouteTableId: routeTableId,
            DestinationCidrBlock: destinationCidrBlock,
            GatewayId: gatewayId
        };
        
        try {
            await this.ec2.createRoute(params).promise();
            console.log(`Route created in ${routeTableId}`);
        } catch (error) {
            console.error('Route creation error:', error);
            throw error;
        }
    }
    
    // Associate Route Table with Subnet
    async associateRouteTable(routeTableId, subnetId) {
        const params = {
            RouteTableId: routeTableId,
            SubnetId: subnetId
        };
        
        try {
            const result = await this.ec2.associateRouteTable(params).promise();
            console.log(`Route table ${routeTableId} associated with subnet ${subnetId}`);
            return result.AssociationId;
        } catch (error) {
            console.error('Route table association error:', error);
            throw error;
        }
    }
    
    // Create Security Group
    async createSecurityGroup(vpcId, groupName, description) {
        const params = {
            GroupName: groupName,
            Description: description,
            VpcId: vpcId,
            TagSpecifications: [{
                ResourceType: 'security-group',
                Tags: [{ Key: 'Name', Value: groupName }]
            }]
        };
        
        try {
            const result = await this.ec2.createSecurityGroup(params).promise();
            console.log(`Security Group ${result.GroupId} created`);
            return result.GroupId;
        } catch (error) {
            console.error('Security group creation error:', error);
            throw error;
        }
    }
    
    // Add Security Group Rule
    async addSecurityGroupRule(groupId, ruleType, protocol, port, cidr) {
        const params = {
            GroupId: groupId,
            [ruleType]: [{
                IpProtocol: protocol,
                FromPort: port,
                ToPort: port,
                CidrIp: cidr
            }]
        };
        
        try {
            await this.ec2.authorizeSecurityGroupIngress(params).promise();
            console.log(`Security group rule added to ${groupId}`);
        } catch (error) {
            console.error('Security group rule error:', error);
            throw error;
        }
    }
}

// Complete VPC Setup
async function setupCompleteVPC() {
    const vpcManager = new VPCManager();
    
    try {
        // 1. Create VPC
        const vpc = await vpcManager.createVPC('MyApp-VPC', '10.0.0.0/16');
        const vpcId = vpc.VpcId;
        
        // 2. Create Internet Gateway
        const igw = await vpcManager.createInternetGateway('MyApp-IGW');
        const igwId = igw.InternetGatewayId;
        
        // 3. Attach IGW to VPC
        await vpcManager.attachInternetGateway(vpcId, igwId);
        
        // 4. Create Public Subnets
        const publicSubnet1 = await vpcManager.createSubnet(
            vpcId, 
            '10.0.1.0/24', 
            'us-east-1a', 
            'Public-Subnet-1'
        );
        
        const publicSubnet2 = await vpcManager.createSubnet(
            vpcId, 
            '10.0.2.0/24', 
            'us-east-1b', 
            'Public-Subnet-2'
        );
        
        // 5. Create Private Subnets
        const privateSubnet1 = await vpcManager.createSubnet(
            vpcId, 
            '10.0.3.0/24', 
            'us-east-1a', 
            'Private-Subnet-1'
        );
        
        const privateSubnet2 = await vpcManager.createSubnet(
            vpcId, 
            '10.0.4.0/24', 
            'us-east-1b', 
            'Private-Subnet-2'
        );
        
        // 6. Create Route Tables
        const publicRouteTable = await vpcManager.createRouteTable(vpcId, 'Public-Route-Table');
        const privateRouteTable = await vpcManager.createRouteTable(vpcId, 'Private-Route-Table');
        
        // 7. Add Routes
        await vpcManager.createRoute(publicRouteTable.RouteTableId, '0.0.0.0/0', igwId);
        
        // 8. Associate Route Tables
        await vpcManager.associateRouteTable(publicRouteTable.RouteTableId, publicSubnet1.SubnetId);
        await vpcManager.associateRouteTable(publicRouteTable.RouteTableId, publicSubnet2.SubnetId);
        await vpcManager.associateRouteTable(privateRouteTable.RouteTableId, privateSubnet1.SubnetId);
        await vpcManager.associateRouteTable(privateRouteTable.RouteTableId, privateSubnet2.SubnetId);
        
        // 9. Create Security Groups
        const webSecurityGroup = await vpcManager.createSecurityGroup(
            vpcId, 
            'Web-Security-Group', 
            'Security group for web servers'
        );
        
        const dbSecurityGroup = await vpcManager.createSecurityGroup(
            vpcId, 
            'DB-Security-Group', 
            'Security group for database servers'
        );
        
        // 10. Add Security Group Rules
        await vpcManager.addSecurityGroupRule(webSecurityGroup, 'IpPermissions', 'tcp', 80, '0.0.0.0/0');
        await vpcManager.addSecurityGroupRule(webSecurityGroup, 'IpPermissions', 'tcp', 443, '0.0.0.0/0');
        await vpcManager.addSecurityGroupRule(webSecurityGroup, 'IpPermissions', 'tcp', 22, '0.0.0.0/0');
        
        await vpcManager.addSecurityGroupRule(dbSecurityGroup, 'IpPermissions', 'tcp', 3306, '10.0.0.0/16');
        
        console.log('Complete VPC setup finished successfully');
        
        return {
            vpcId,
            publicSubnets: [publicSubnet1.SubnetId, publicSubnet2.SubnetId],
            privateSubnets: [privateSubnet1.SubnetId, privateSubnet2.SubnetId],
            securityGroups: {
                web: webSecurityGroup,
                database: dbSecurityGroup
            }
        };
    } catch (error) {
        console.error('VPC setup error:', error);
        throw error;
    }
}
```

---

## IAM & Security

### Q7. What is IAM and how do you implement security best practices?
**Answer:** AWS Identity and Access Management (IAM) controls access to AWS services and resources through users, groups, roles, and policies.

**Key Components:**
- **Users:** Individual AWS accounts
- **Groups:** Collections of users
- **Roles:** Temporary credentials for services
- **Policies:** JSON documents defining permissions
- **MFA:** Multi-factor authentication
- **Access Keys:** Programmatic access credentials

**Real-time Example:**
```javascript
// IAM Security Management System
const AWS = require('aws-sdk');
const iam = new AWS.IAM();

class IAMManager {
    constructor() {
        this.iam = iam;
    }
    
    // Create IAM User
    async createUser(username, path = '/') {
        const params = {
            UserName: username,
            Path: path,
            Tags: [
                { Key: 'Environment', Value: 'production' },
                { Key: 'Department', Value: 'Engineering' }
            ]
        };
        
        try {
            const result = await this.iam.createUser(params).promise();
            console.log(`User ${username} created`);
            return result.User;
        } catch (error) {
            console.error('User creation error:', error);
            throw error;
        }
    }
    
    // Create IAM Group
    async createGroup(groupName, path = '/') {
        const params = {
            GroupName: groupName,
            Path: path
        };
        
        try {
            const result = await this.iam.createGroup(params).promise();
            console.log(`Group ${groupName} created`);
            return result.Group;
        } catch (error) {
            console.error('Group creation error:', error);
            throw error;
        }
    }
    
    // Add User to Group
    async addUserToGroup(username, groupName) {
        const params = {
            UserName: username,
            GroupName: groupName
        };
        
        try {
            await this.iam.addUserToGroup(params).promise();
            console.log(`User ${username} added to group ${groupName}`);
        } catch (error) {
            console.error('Add user to group error:', error);
            throw error;
        }
    }
    
    // Create IAM Role
    async createRole(roleName, assumeRolePolicyDocument) {
        const params = {
            RoleName: roleName,
            AssumeRolePolicyDocument: JSON.stringify(assumeRolePolicyDocument),
            Description: `Role for ${roleName}`,
            Tags: [
                { Key: 'Environment', Value: 'production' }
            ]
        };
        
        try {
            const result = await this.iam.createRole(params).promise();
            console.log(`Role ${roleName} created`);
            return result.Role;
        } catch (error) {
            console.error('Role creation error:', error);
            throw error;
        }
    }
    
    // Attach Policy to Role
    async attachRolePolicy(roleName, policyArn) {
        const params = {
            RoleName: roleName,
            PolicyArn: policyArn
        };
        
        try {
            await this.iam.attachRolePolicy(params).promise();
            console.log(`Policy ${policyArn} attached to role ${roleName}`);
        } catch (error) {
            console.error('Attach policy error:', error);
            throw error;
        }
    }
    
    // Create Custom Policy
    async createPolicy(policyName, policyDocument, description) {
        const params = {
            PolicyName: policyName,
            PolicyDocument: JSON.stringify(policyDocument),
            Description: description
        };
        
        try {
            const result = await this.iam.createPolicy(params).promise();
            console.log(`Policy ${policyName} created`);
            return result.Policy;
        } catch (error) {
            console.error('Policy creation error:', error);
            throw error;
        }
    }
    
    // Create Access Key
    async createAccessKey(username) {
        const params = {
            UserName: username
        };
        
        try {
            const result = await this.iam.createAccessKey(params).promise();
            console.log(`Access key created for ${username}`);
            return result.AccessKey;
        } catch (error) {
            console.error('Access key creation error:', error);
            throw error;
        }
    }
    
    // Enable MFA for User
    async enableMFA(username, serialNumber, authenticationCode1, authenticationCode2) {
        const params = {
            UserName: username,
            SerialNumber: serialNumber,
            AuthenticationCode1: authenticationCode1,
            AuthenticationCode2: authenticationCode2
        };
        
        try {
            await this.iam.enableMFADevice(params).promise();
            console.log(`MFA enabled for ${username}`);
        } catch (error) {
            console.error('MFA enable error:', error);
            throw error;
        }
    }
}

// Security Best Practices Implementation
class SecurityManager {
    constructor() {
        this.iam = new IAMManager();
    }
    
    // Implement least privilege access
    async setupLeastPrivilegeAccess() {
        // Create developer policy with minimal permissions
        const developerPolicy = {
            Version: '2012-10-17',
            Statement: [
                {
                    Effect: 'Allow',
                    Action: [
                        's3:GetObject',
                        's3:PutObject'
                    ],
                    Resource: 'arn:aws:s3:::myapp-dev-bucket/*'
                },
                {
                    Effect: 'Allow',
                    Action: [
                        'dynamodb:GetItem',
                        'dynamodb:PutItem',
                        'dynamodb:Query'
                    ],
                    Resource: 'arn:aws:dynamodb:us-east-1:123456789012:table/MyApp-Dev-*'
                }
            ]
        };
        
        await this.iam.createPolicy(
            'DeveloperPolicy',
            developerPolicy,
            'Minimal permissions for developers'
        );
    }
    
    // Setup cross-account access
    async setupCrossAccountAccess(trustedAccountId) {
        const trustPolicy = {
            Version: '2012-10-17',
            Statement: [
                {
                    Effect: 'Allow',
                    Principal: {
                        AWS: `arn:aws:iam::${trustedAccountId}:root`
                    },
                    Action: 'sts:AssumeRole',
                    Condition: {
                        StringEquals: {
                            'sts:ExternalId': 'unique-external-id'
                        }
                    }
                }
            ]
        };
        
        await this.iam.createRole('CrossAccountRole', trustPolicy);
    }
    
    // Implement resource-based policies
    async setupResourceBasedPolicies() {
        // S3 bucket policy
        const bucketPolicy = {
            Version: '2012-10-17',
            Statement: [
                {
                    Effect: 'Allow',
                    Principal: {
                        AWS: 'arn:aws:iam::123456789012:user/developer'
                    },
                    Action: 's3:GetObject',
                    Resource: 'arn:aws:s3:::myapp-bucket/*',
                    Condition: {
                        IpAddress: {
                            'aws:SourceIp': '203.0.113.0/24'
                        }
                    }
                }
            ]
        };
        
        return bucketPolicy;
    }
}

// Usage Examples
const iamManager = new IAMManager();
const securityManager = new SecurityManager();

// Complete IAM setup
async function setupCompleteIAM() {
    try {
        // 1. Create users
        await iamManager.createUser('developer1');
        await iamManager.createUser('developer2');
        await iamManager.createUser('admin');
        
        // 2. Create groups
        await iamManager.createGroup('Developers');
        await iamManager.createGroup('Admins');
        
        // 3. Add users to groups
        await iamManager.addUserToGroup('developer1', 'Developers');
        await iamManager.addUserToGroup('developer2', 'Developers');
        await iamManager.addUserToGroup('admin', 'Admins');
        
        // 4. Create roles
        const lambdaRole = await iamManager.createRole('LambdaExecutionRole', {
            Version: '2012-10-17',
            Statement: [
                {
                    Effect: 'Allow',
                    Principal: {
                        Service: 'lambda.amazonaws.com'
                    },
                    Action: 'sts:AssumeRole'
                }
            ]
        });
        
        // 5. Attach policies to roles
        await iamManager.attachRolePolicy('LambdaExecutionRole', 'arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole');
        
        // 6. Setup security best practices
        await securityManager.setupLeastPrivilegeAccess();
        await securityManager.setupCrossAccountAccess('987654321098');
        
        console.log('Complete IAM setup finished');
    } catch (error) {
        console.error('IAM setup error:', error);
        throw error;
    }
}
```

---

## Load Balancing & Auto Scaling

### Q8. How do you implement load balancing and auto scaling?
**Answer:** AWS provides Application Load Balancer (ALB), Network Load Balancer (NLB), and Auto Scaling Groups to distribute traffic and automatically scale resources.

**Key Components:**
- **ALB:** Layer 7 load balancing with advanced routing
- **NLB:** Layer 4 load balancing for high performance
- **Auto Scaling Groups:** Automatic scaling based on metrics
- **Target Groups:** Backend resources for load balancers
- **Launch Templates:** EC2 instance configurations

**Real-time Example:**
```javascript
// Load Balancer and Auto Scaling Management
const AWS = require('aws-sdk');
const elbv2 = new AWS.ELBv2();
const autoscaling = new AWS.AutoScaling();

class LoadBalancerManager {
    constructor() {
        this.elbv2 = elbv2;
        this.autoscaling = autoscaling;
    }
    
    // Create Application Load Balancer
    async createALB(name, subnets, securityGroups) {
        const params = {
            Name: name,
            Subnets: subnets,
            SecurityGroups: securityGroups,
            Scheme: 'internet-facing',
            Type: 'application',
            Tags: [
                { Key: 'Name', Value: name },
                { Key: 'Environment', Value: 'production' }
            ]
        };
        
        try {
            const result = await this.elbv2.createLoadBalancer(params).promise();
            console.log(`ALB ${name} created: ${result.LoadBalancers[0].LoadBalancerArn}`);
            return result.LoadBalancers[0];
        } catch (error) {
            console.error('ALB creation error:', error);
            throw error;
        }
    }
    
    // Create Target Group
    async createTargetGroup(name, vpcId, port, protocol = 'HTTP') {
        const params = {
            Name: name,
            Protocol: protocol,
            Port: port,
            VpcId: vpcId,
            HealthCheckProtocol: protocol,
            HealthCheckPort: port.toString(),
            HealthCheckPath: '/health',
            HealthCheckIntervalSeconds: 30,
            HealthCheckTimeoutSeconds: 5,
            HealthyThresholdCount: 2,
            UnhealthyThresholdCount: 3,
            Tags: [
                { Key: 'Name', Value: name }
            ]
        };
        
        try {
            const result = await this.elbv2.createTargetGroup(params).promise();
            console.log(`Target Group ${name} created: ${result.TargetGroups[0].TargetGroupArn}`);
            return result.TargetGroups[0];
        } catch (error) {
            console.error('Target group creation error:', error);
            throw error;
        }
    }
    
    // Create Listener
    async createListener(loadBalancerArn, targetGroupArn, port = 80, protocol = 'HTTP') {
        const params = {
            DefaultActions: [
                {
                    Type: 'forward',
                    TargetGroupArn: targetGroupArn
                }
            ],
            LoadBalancerArn: loadBalancerArn,
            Port: port,
            Protocol: protocol
        };
        
        try {
            const result = await this.elbv2.createListener(params).promise();
            console.log(`Listener created for ALB`);
            return result.Listeners[0];
        } catch (error) {
            console.error('Listener creation error:', error);
            throw error;
        }
    }
    
    // Register Targets
    async registerTargets(targetGroupArn, targets) {
        const params = {
            TargetGroupArn: targetGroupArn,
            Targets: targets
        };
        
        try {
            await this.elbv2.registerTargets(params).promise();
            console.log(`Targets registered to target group`);
        } catch (error) {
            console.error('Target registration error:', error);
            throw error;
        }
    }
}

class AutoScalingManager {
    constructor() {
        this.autoscaling = autoscaling;
    }
    
    // Create Launch Template
    async createLaunchTemplate(templateName, imageId, instanceType, securityGroups, userData) {
        const params = {
            LaunchTemplateName: templateName,
            LaunchTemplateData: {
                ImageId: imageId,
                InstanceType: instanceType,
                SecurityGroupIds: securityGroups,
                UserData: Buffer.from(userData).toString('base64'),
                TagSpecifications: [
                    {
                        ResourceType: 'instance',
                        Tags: [
                            { Key: 'Name', Value: `${templateName}-instance` },
                            { Key: 'Environment', Value: 'production' }
                        ]
                    }
                ]
            }
        };
        
        try {
            const result = await this.autoscaling.createLaunchTemplate(params).promise();
            console.log(`Launch template ${templateName} created`);
            return result.LaunchTemplate;
        } catch (error) {
            console.error('Launch template creation error:', error);
            throw error;
        }
    }
    
    // Create Auto Scaling Group
    async createAutoScalingGroup(groupName, launchTemplateName, targetGroupArn, subnets, minSize, maxSize, desiredCapacity) {
        const params = {
            AutoScalingGroupName: groupName,
            LaunchTemplate: {
                LaunchTemplateName: launchTemplateName,
                Version: '$Latest'
            },
            TargetGroupARNs: [targetGroupArn],
            VPCZoneIdentifier: subnets.join(','),
            MinSize: minSize,
            MaxSize: maxSize,
            DesiredCapacity: desiredCapacity,
            HealthCheckType: 'ELB',
            HealthCheckGracePeriod: 300,
            Tags: [
                { Key: 'Name', Value: groupName },
                { Key: 'Environment', Value: 'production' }
            ]
        };
        
        try {
            const result = await this.autoscaling.createAutoScalingGroup(params).promise();
            console.log(`Auto Scaling Group ${groupName} created`);
            return result;
        } catch (error) {
            console.error('Auto Scaling Group creation error:', error);
            throw error;
        }
    }
    
    // Create Scaling Policy
    async createScalingPolicy(groupName, policyName, adjustmentType, scalingAdjustment, cooldown = 300) {
        const params = {
            AutoScalingGroupName: groupName,
            PolicyName: policyName,
            AdjustmentType: adjustmentType,
            ScalingAdjustment: scalingAdjustment,
            Cooldown: cooldown
        };
        
        try {
            const result = await this.autoscaling.putScalingPolicy(params).promise();
            console.log(`Scaling policy ${policyName} created`);
            return result;
        } catch (error) {
            console.error('Scaling policy creation error:', error);
            throw error;
        }
    }
    
    // Create CloudWatch Alarm for Scaling
    async createScalingAlarm(alarmName, groupName, policyArn, metricName, threshold) {
        const cloudwatch = new AWS.CloudWatch();
        
        const params = {
            AlarmName: alarmName,
            ComparisonOperator: 'GreaterThanThreshold',
            EvaluationPeriods: 2,
            MetricName: metricName,
            Namespace: 'AWS/AutoScaling',
            Period: 300,
            Statistic: 'Average',
            Threshold: threshold,
            ActionsEnabled: true,
            AlarmActions: [policyArn],
            AlarmDescription: `Auto scaling alarm for ${groupName}`
        };
        
        try {
            await cloudwatch.putMetricAlarm(params).promise();
            console.log(`Scaling alarm ${alarmName} created`);
        } catch (error) {
            console.error('Scaling alarm creation error:', error);
            throw error;
        }
    }
}

// Complete Load Balancing and Auto Scaling Setup
async function setupLoadBalancingAndAutoScaling() {
    const lbManager = new LoadBalancerManager();
    const asManager = new AutoScalingManager();
    
    try {
        // 1. Create Application Load Balancer
        const alb = await lbManager.createALB(
            'MyApp-ALB',
            ['subnet-12345678', 'subnet-87654321'],
            ['sg-12345678']
        );
        
        // 2. Create Target Groups
        const webTargetGroup = await lbManager.createTargetGroup(
            'Web-Target-Group',
            'vpc-12345678',
            3000
        );
        
        const apiTargetGroup = await lbManager.createTargetGroup(
            'API-Target-Group',
            'vpc-12345678',
            8000
        );
        
        // 3. Create Listeners
        await lbManager.createListener(alb.LoadBalancerArn, webTargetGroup.TargetGroupArn, 80);
        await lbManager.createListener(alb.LoadBalancerArn, apiTargetGroup.TargetGroupArn, 8080);
        
        // 4. Create Launch Templates
        const webLaunchTemplate = await asManager.createLaunchTemplate(
            'WebLaunchTemplate',
            'ami-0c55b159cbfafe1f0',
            't3.micro',
            ['sg-12345678'],
            `#!/bin/bash
            yum update -y
            yum install -y nodejs
            npm install -g pm2
            # Application setup code here
            `
        );
        
        const apiLaunchTemplate = await asManager.createLaunchTemplate(
            'APILaunchTemplate',
            'ami-0c55b159cbfafe1f0',
            't3.small',
            ['sg-87654321'],
            `#!/bin/bash
            yum update -y
            yum install -y nodejs
            npm install -g pm2
            # API setup code here
            `
        );
        
        // 5. Create Auto Scaling Groups
        const webASG = await asManager.createAutoScalingGroup(
            'Web-ASG',
            'WebLaunchTemplate',
            webTargetGroup.TargetGroupArn,
            ['subnet-12345678', 'subnet-87654321'],
            2, 10, 3
        );
        
        const apiASG = await asManager.createAutoScalingGroup(
            'API-ASG',
            'APILaunchTemplate',
            apiTargetGroup.TargetGroupArn,
            ['subnet-12345678', 'subnet-87654321'],
            1, 5, 2
        );
        
        // 6. Create Scaling Policies
        const webScaleUpPolicy = await asManager.createScalingPolicy(
            'Web-ASG',
            'WebScaleUpPolicy',
            'ChangeInCapacity',
            1
        );
        
        const webScaleDownPolicy = await asManager.createScalingPolicy(
            'Web-ASG',
            'WebScaleDownPolicy',
            'ChangeInCapacity',
            -1
        );
        
        // 7. Create CloudWatch Alarms
        await asManager.createScalingAlarm(
            'WebHighCPUAlarm',
            'Web-ASG',
            webScaleUpPolicy.PolicyARN,
            'CPUUtilization',
            70
        );
        
        await asManager.createScalingAlarm(
            'WebLowCPUAlarm',
            'Web-ASG',
            webScaleDownPolicy.PolicyARN,
            'CPUUtilization',
            30
        );
        
        console.log('Load balancing and auto scaling setup completed');
        
        return {
            loadBalancer: alb,
            targetGroups: [webTargetGroup, apiTargetGroup],
            autoScalingGroups: [webASG, apiASG]
        };
    } catch (error) {
        console.error('Setup error:', error);
        throw error;
    }
}
```

---

## CloudFront & CDN

### Q9. How do you implement CloudFront for global content delivery?
**Answer:** Amazon CloudFront is a global CDN that delivers content with low latency and high transfer speeds.

**Key Features:**
- Global edge locations
- SSL/TLS termination
- Custom error pages
- Origin access identity
- Cache behaviors
- Real-time metrics

**Real-time Example:**
```javascript
// CloudFront Distribution Management
const AWS = require('aws-sdk');
const cloudfront = new AWS.CloudFront();

class CloudFrontManager {
    constructor() {
        this.cloudfront = cloudfront;
    }
    
    // Create CloudFront Distribution
    async createDistribution(domainName, s3BucketName) {
        const params = {
            DistributionConfig: {
                CallerReference: `myapp-${Date.now()}`,
                Comment: 'MyApp CloudFront Distribution',
                DefaultCacheBehavior: {
                    TargetOriginId: 'S3-MyApp-Bucket',
                    ViewerProtocolPolicy: 'redirect-to-https',
                    TrustedSigners: {
                        Enabled: false,
                        Quantity: 0
                    },
                    ForwardedValues: {
                        QueryString: false,
                        Cookies: {
                            Forward: 'none'
                        }
                    },
                    MinTTL: 0,
                    DefaultTTL: 86400,
                    MaxTTL: 31536000
                },
                Origins: {
                    Quantity: 1,
                    Items: [
                        {
                            Id: 'S3-MyApp-Bucket',
                            DomainName: `${s3BucketName}.s3.amazonaws.com`,
                            S3OriginConfig: {
                                OriginAccessIdentity: ''
                            }
                        }
                    ]
                },
                Enabled: true,
                PriceClass: 'PriceClass_100',
                HttpVersion: 'http2',
                CacheBehaviors: {
                    Quantity: 1,
                    Items: [
                        {
                            PathPattern: '/api/*',
                            TargetOriginId: 'S3-MyApp-Bucket',
                            ViewerProtocolPolicy: 'redirect-to-https',
                            TrustedSigners: {
                                Enabled: false,
                                Quantity: 0
                            },
                            ForwardedValues: {
                                QueryString: true,
                                Headers: {
                                    Quantity: 2,
                                    Items: ['Authorization', 'Content-Type']
                                },
                                Cookies: {
                                    Forward: 'none'
                                }
                            },
                            MinTTL: 0,
                            DefaultTTL: 0,
                            MaxTTL: 0
                        }
                    ]
                },
                CustomErrorResponses: {
                    Quantity: 2,
                    Items: [
                        {
                            ErrorCode: 404,
                            ResponsePagePath: '/404.html',
                            ResponseCode: '404',
                            ErrorCachingMinTTL: 300
                        },
                        {
                            ErrorCode: 403,
                            ResponsePagePath: '/403.html',
                            ResponseCode: '403',
                            ErrorCachingMinTTL: 300
                        }
                    ]
                },
                Aliases: {
                    Quantity: 1,
                    Items: [domainName]
                },
                DefaultRootObject: 'index.html'
            }
        };
        
        try {
            const result = await this.cloudfront.createDistribution(params).promise();
            console.log(`CloudFront distribution created: ${result.Distribution.Id}`);
            return result.Distribution;
        } catch (error) {
            console.error('CloudFront creation error:', error);
            throw error;
        }
    }
    
    // Create Invalidation
    async createInvalidation(distributionId, paths) {
        const params = {
            DistributionId: distributionId,
            InvalidationBatch: {
                CallerReference: `invalidation-${Date.now()}`,
                Paths: {
                    Quantity: paths.length,
                    Items: paths
                }
            }
        };
        
        try {
            const result = await this.cloudfront.createInvalidation(params).promise();
            console.log(`Invalidation created: ${result.Invalidation.Id}`);
            return result.Invalidation;
        } catch (error) {
            console.error('Invalidation creation error:', error);
            throw error;
        }
    }
    
    // Get Distribution
    async getDistribution(distributionId) {
        const params = {
            Id: distributionId
        };
        
        try {
            const result = await this.cloudfront.getDistribution(params).promise();
            return result.Distribution;
        } catch (error) {
            console.error('Get distribution error:', error);
            throw error;
        }
    }
    
    // Update Distribution
    async updateDistribution(distributionId, etag, config) {
        const params = {
            Id: distributionId,
            IfMatch: etag,
            DistributionConfig: config
        };
        
        try {
            const result = await this.cloudfront.updateDistribution(params).promise();
            console.log(`Distribution ${distributionId} updated`);
            return result.Distribution;
        } catch (error) {
            console.error('Distribution update error:', error);
            throw error;
        }
    }
}

// CloudFront with S3 Integration
class CloudFrontS3Integration {
    constructor() {
        this.cloudfront = new CloudFrontManager();
        this.s3 = new AWS.S3();
    }
    
    // Setup CloudFront with S3
    async setupCloudFrontWithS3(domainName, bucketName) {
        try {
            // 1. Create S3 bucket
            await this.s3.createBucket({
                Bucket: bucketName,
                ACL: 'private'
            }).promise();
            
            // 2. Configure bucket for CloudFront
            const bucketPolicy = {
                Version: '2012-10-17',
                Statement: [
                    {
                        Sid: 'AllowCloudFrontServicePrincipal',
                        Effect: 'Allow',
                        Principal: {
                            Service: 'cloudfront.amazonaws.com'
                        },
                        Action: 's3:GetObject',
                        Resource: `arn:aws:s3:::${bucketName}/*`,
                        Condition: {
                            StringEquals: {
                                'AWS:SourceArn': `arn:aws:cloudfront::${process.env.AWS_ACCOUNT_ID}:distribution/*`
                            }
                        }
                    }
                ]
            };
            
            await this.s3.putBucketPolicy({
                Bucket: bucketName,
                Policy: JSON.stringify(bucketPolicy)
            }).promise();
            
            // 3. Create CloudFront distribution
            const distribution = await this.cloudfront.createDistribution(domainName, bucketName);
            
            return distribution;
        } catch (error) {
            console.error('CloudFront S3 setup error:', error);
            throw error;
        }
    }
    
    // Upload and sync content
    async uploadContent(bucketName, key, content, contentType) {
        const params = {
            Bucket: bucketName,
            Key: key,
            Body: content,
            ContentType: contentType,
            CacheControl: 'max-age=31536000' // 1 year
        };
        
        try {
            await this.s3.putObject(params).promise();
            console.log(`Content uploaded: ${key}`);
        } catch (error) {
            console.error('Content upload error:', error);
            throw error;
        }
    }
    
    // Invalidate cache
    async invalidateCache(distributionId, paths) {
        return await this.cloudfront.createInvalidation(distributionId, paths);
    }
}

// Usage
const cfManager = new CloudFrontManager();
const cfS3Integration = new CloudFrontS3Integration();

// Setup complete CloudFront distribution
async function setupCloudFrontDistribution() {
    try {
        const distribution = await cfS3Integration.setupCloudFrontWithS3(
            'myapp.example.com',
            'myapp-static-assets'
        );
        
        // Upload static assets
        await cfS3Integration.uploadContent(
            'myapp-static-assets',
            'index.html',
            '<html><body>Hello World!</body></html>',
            'text/html'
        );
        
        // Upload CSS
        await cfS3Integration.uploadContent(
            'myapp-static-assets',
            'styles.css',
            'body { font-family: Arial; }',
            'text/css'
        );
        
        // Upload JavaScript
        await cfS3Integration.uploadContent(
            'myapp-static-assets',
            'app.js',
            'console.log("Hello from CloudFront!");',
            'application/javascript'
        );
        
        // Invalidate cache for updated content
        await cfS3Integration.invalidateCache(distribution.Id, ['/index.html', '/styles.css']);
        
        console.log('CloudFront distribution setup completed');
        return distribution;
    } catch (error) {
        console.error('CloudFront setup error:', error);
        throw error;
    }
}
```

---

## Deployment & DevOps

### Q10. How do you implement CI/CD pipelines and deployment strategies?
**Answer:** AWS provides CodePipeline, CodeBuild, and CodeDeploy for implementing CI/CD pipelines with automated testing, building, and deployment.

**Key Services:**
- **CodePipeline:** Orchestrates CI/CD workflows
- **CodeBuild:** Builds and tests code
- **CodeDeploy:** Deploys applications
- **CodeCommit:** Git repository service
- **ECS/EKS:** Container orchestration
- **Elastic Beanstalk:** Platform-as-a-Service

**Real-time Example:**
```javascript
// CI/CD Pipeline Management
const AWS = require('aws-sdk');
const codepipeline = new AWS.CodePipeline();
const codebuild = new AWS.CodeBuild();
const codedeploy = new AWS.CodeDeploy();

class CICDManager {
    constructor() {
        this.codepipeline = codepipeline;
        this.codebuild = codebuild;
        this.codedeploy = codedeploy;
    }
    
    // Create CodeBuild Project
    async createCodeBuildProject(projectName, sourceRepo, buildSpec) {
        const params = {
            name: projectName,
            description: `Build project for ${projectName}`,
            source: {
                type: 'GITHUB',
                location: sourceRepo,
                buildspec: buildSpec
            },
            artifacts: {
                type: 'CODEPIPELINE'
            },
            environment: {
                type: 'LINUX_CONTAINER',
                image: 'aws/codebuild/amazonlinux2-x86_64-standard:4.0',
                computeType: 'BUILD_GENERAL1_SMALL',
                environmentVariables: [
                    {
                        name: 'NODE_ENV',
                        value: 'production'
                    }
                ]
            },
            serviceRole: process.env.CODEBUILD_ROLE_ARN,
            tags: [
                { key: 'Environment', value: 'production' },
                { key: 'Project', value: projectName }
            ]
        };
        
        try {
            const result = await this.codebuild.createProject(params).promise();
            console.log(`CodeBuild project ${projectName} created`);
            return result.project;
        } catch (error) {
            console.error('CodeBuild project creation error:', error);
            throw error;
        }
    }
    
    // Create CodePipeline
    async createCodePipeline(pipelineName, sourceRepo, buildProjectName) {
        const params = {
            pipeline: {
                name: pipelineName,
                roleArn: process.env.CODEPIPELINE_ROLE_ARN,
                artifactStore: {
                    type: 'S3',
                    location: process.env.ARTIFACT_BUCKET
                },
                stages: [
                    {
                        name: 'Source',
                        actions: [
                            {
                                name: 'SourceAction',
                                actionTypeId: {
                                    category: 'Source',
                                    owner: 'AWS',
                                    provider: 'GitHub',
                                    version: '1'
                                },
                                configuration: {
                                    Owner: process.env.GITHUB_OWNER,
                                    Repo: sourceRepo,
                                    Branch: 'main',
                                    OAuthToken: process.env.GITHUB_TOKEN
                                },
                                outputArtifacts: [
                                    { name: 'SourceOutput' }
                                ]
                            }
                        ]
                    },
                    {
                        name: 'Build',
                        actions: [
                            {
                                name: 'BuildAction',
                                actionTypeId: {
                                    category: 'Build',
                                    owner: 'AWS',
                                    provider: 'CodeBuild',
                                    version: '1'
                                },
                                configuration: {
                                    ProjectName: buildProjectName
                                },
                                inputArtifacts: [
                                    { name: 'SourceOutput' }
                                ],
                                outputArtifacts: [
                                    { name: 'BuildOutput' }
                                ]
                            }
                        ]
                    },
                    {
                        name: 'Deploy',
                        actions: [
                            {
                                name: 'DeployAction',
                                actionTypeId: {
                                    category: 'Deploy',
                                    owner: 'AWS',
                                    provider: 'ECS',
                                    version: '1'
                                },
                                configuration: {
                                    ClusterName: process.env.ECS_CLUSTER,
                                    ServiceName: process.env.ECS_SERVICE
                                },
                                inputArtifacts: [
                                    { name: 'BuildOutput' }
                                ]
                            }
                        ]
                    }
                ]
            }
        };
        
        try {
            const result = await this.codepipeline.createPipeline(params).promise();
            console.log(`CodePipeline ${pipelineName} created`);
            return result.pipeline;
        } catch (error) {
            console.error('CodePipeline creation error:', error);
            throw error;
        }
    }
    
    // Create Deployment Group
    async createDeploymentGroup(appName, groupName, ec2TagFilters) {
        const params = {
            applicationName: appName,
            deploymentGroupName: groupName,
            serviceRoleArn: process.env.CODEDEPLOY_ROLE_ARN,
            ec2TagFilters: ec2TagFilters,
            deploymentConfigName: 'CodeDeployDefault.OneAtATime',
            autoRollbackConfiguration: {
                enabled: true,
                events: ['DEPLOYMENT_FAILURE']
            }
        };
        
        try {
            const result = await this.codedeploy.createDeploymentGroup(params).promise();
            console.log(`Deployment group ${groupName} created`);
            return result;
        } catch (error) {
            console.error('Deployment group creation error:', error);
            throw error;
        }
    }
}

// Build Specification Examples
const buildSpecs = {
    // Node.js Application Build Spec
    nodejs: `version: 0.2
phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
  build:
    commands:
      - echo Build started on \`date\`
      - echo Building the Docker image...
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
  post_build:
    commands:
      - echo Build completed on \`date\`
      - echo Pushing the Docker image...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
artifacts:
  files:
    - '**/*'
  name: build-output-$(date +%Y-%m-%d)
cache:
  paths:
    - '/root/.npm/**/*'`,

    // React Application Build Spec
    react: `version: 0.2
phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - echo Installing dependencies...
      - npm ci
  pre_build:
    commands:
      - echo Running tests...
      - npm run test
  build:
    commands:
      - echo Building application...
      - npm run build
  post_build:
    commands:
      - echo Build completed on \`date\`
artifacts:
  files:
    - 'build/**/*'
  name: react-build-$(date +%Y-%m-%d)`
};

// Docker Deployment Configuration
const dockerConfig = {
    // Dockerfile for Node.js application
    dockerfile: `FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

USER node

CMD ["npm", "start"]`,

    // docker-compose.yml for local development
    dockerCompose: `version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DB_HOST=db
    depends_on:
      - db
  
  db:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=rootpassword
      - MYSQL_DATABASE=myapp
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:`
};

// ECS Task Definition
const ecsTaskDefinition = {
    family: 'myapp-task',
    networkMode: 'awsvpc',
    requiresCompatibilities: ['FARGATE'],
    cpu: '256',
    memory: '512',
    executionRoleArn: 'arn:aws:iam::123456789012:role/ecsTaskExecutionRole',
    taskRoleArn: 'arn:aws:iam::123456789012:role/ecsTaskRole',
    containerDefinitions: [
        {
            name: 'myapp-container',
            image: '123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest',
            portMappings: [
                {
                    containerPort: 3000,
                    protocol: 'tcp'
                }
            ],
            environment: [
                {
                    name: 'NODE_ENV',
                    value: 'production'
                }
            ],
            logConfiguration: {
                logDriver: 'awslogs',
                options: {
                    'awslogs-group': '/ecs/myapp',
                    'awslogs-region': 'us-east-1',
                    'awslogs-stream-prefix': 'ecs'
                }
            }
        }
    ]
};

// Complete CI/CD Setup
async function setupCompleteCICD() {
    const cicdManager = new CICDManager();
    
    try {
        // 1. Create CodeBuild projects
        const buildProject = await cicdManager.createCodeBuildProject(
            'MyApp-Build',
            'myorg/myapp',
            buildSpecs.nodejs
        );
        
        // 2. Create CodePipeline
        const pipeline = await cicdManager.createCodePipeline(
            'MyApp-Pipeline',
            'myapp',
            'MyApp-Build'
        );
        
        // 3. Create deployment group
        await cicdManager.createDeploymentGroup(
            'MyApp',
            'Production',
            [
                {
                    Key: 'Environment',
                    Value: 'Production',
                    Type: 'KEY_AND_VALUE'
                }
            ]
        );
        
        console.log('Complete CI/CD pipeline setup finished');
        return {
            pipeline,
            buildProject
        };
    } catch (error) {
        console.error('CI/CD setup error:', error);
        throw error;
    }
}
```

---

## Monitoring & Logging

### Q11. How do you implement monitoring and logging with CloudWatch?
**Answer:** Amazon CloudWatch provides monitoring, logging, and alerting for AWS resources and applications.

**Key Features:**
- **Metrics:** Performance and operational data
- **Logs:** Centralized log management
- **Alarms:** Automated responses
- **Dashboards:** Visual monitoring
- **Insights:** Log analysis

**Real-time Example:**
```javascript
// CloudWatch Monitoring and Logging System
const AWS = require('aws-sdk');
const cloudwatch = new AWS.CloudWatch();
const cloudwatchlogs = new AWS.CloudWatchLogs();

class CloudWatchManager {
    constructor() {
        this.cloudwatch = cloudwatch;
        this.cloudwatchlogs = cloudwatchlogs;
    }
    
    // Create Log Group
    async createLogGroup(logGroupName, retentionInDays = 30) {
        const params = {
            logGroupName: logGroupName,
            retentionInDays: retentionInDays
        };
        
        try {
            await this.cloudwatchlogs.createLogGroup(params).promise();
            console.log(`Log group ${logGroupName} created`);
        } catch (error) {
            if (error.code !== 'ResourceAlreadyExistsException') {
                console.error('Log group creation error:', error);
                throw error;
            }
        }
    }
    
    // Create Log Stream
    async createLogStream(logGroupName, logStreamName) {
        const params = {
            logGroupName: logGroupName,
            logStreamName: logStreamName
        };
        
        try {
            await this.cloudwatchlogs.createLogStream(params).promise();
            console.log(`Log stream ${logStreamName} created`);
        } catch (error) {
            if (error.code !== 'ResourceAlreadyExistsException') {
                console.error('Log stream creation error:', error);
                throw error;
            }
        }
    }
    
    // Put Log Events
    async putLogEvents(logGroupName, logStreamName, logEvents) {
        const params = {
            logGroupName: logGroupName,
            logStreamName: logStreamName,
            logEvents: logEvents
        };
        
        try {
            await this.cloudwatchlogs.putLogEvents(params).promise();
            console.log(`Log events sent to ${logStreamName}`);
        } catch (error) {
            console.error('Put log events error:', error);
            throw error;
        }
    }
    
    // Create Custom Metric
    async putMetricData(namespace, metricName, value, unit = 'Count') {
        const params = {
            Namespace: namespace,
            MetricData: [
                {
                    MetricName: metricName,
                    Value: value,
                    Unit: unit,
                    Timestamp: new Date()
                }
            ]
        };
        
        try {
            await this.cloudwatch.putMetricData(params).promise();
            console.log(`Custom metric ${metricName} sent`);
        } catch (error) {
            console.error('Put metric data error:', error);
            throw error;
        }
    }
    
    // Create CloudWatch Alarm
    async createAlarm(alarmName, metricName, namespace, threshold, comparisonOperator = 'GreaterThanThreshold') {
        const params = {
            AlarmName: alarmName,
            ComparisonOperator: comparisonOperator,
            EvaluationPeriods: 2,
            MetricName: metricName,
            Namespace: namespace,
            Period: 300,
            Statistic: 'Average',
            Threshold: threshold,
            ActionsEnabled: true,
            AlarmActions: [process.env.SNS_TOPIC_ARN],
            AlarmDescription: `Alarm for ${metricName}`,
            TreatMissingData: 'notBreaching'
        };
        
        try {
            await this.cloudwatch.putMetricAlarm(params).promise();
            console.log(`Alarm ${alarmName} created`);
        } catch (error) {
            console.error('Alarm creation error:', error);
            throw error;
        }
    }
    
    // Create Dashboard
    async createDashboard(dashboardName, widgets) {
        const params = {
            DashboardName: dashboardName,
            DashboardBody: JSON.stringify({
                widgets: widgets
            })
        };
        
        try {
            await this.cloudwatch.putDashboard(params).promise();
            console.log(`Dashboard ${dashboardName} created`);
        } catch (error) {
            console.error('Dashboard creation error:', error);
            throw error;
        }
    }
    
    // Query Logs with Insights
    async queryLogs(logGroupName, query, startTime, endTime) {
        const params = {
            logGroupName: logGroupName,
            startTime: startTime,
            endTime: endTime,
            queryString: query
        };
        
        try {
            const result = await this.cloudwatchlogs.startQuery(params).promise();
            return result.queryId;
        } catch (error) {
            console.error('Query logs error:', error);
            throw error;
        }
    }
}

// Application Monitoring Implementation
class ApplicationMonitor {
    constructor() {
        this.cloudwatch = new CloudWatchManager();
    }
    
    // Log application events
    async logApplicationEvent(level, message, metadata = {}) {
        const logEvent = {
            timestamp: Date.now(),
            message: JSON.stringify({
                level,
                message,
                metadata,
                timestamp: new Date().toISOString()
            })
        };
        
        await this.cloudwatch.putLogEvents(
            '/aws/lambda/myapp',
            'application-logs',
            [logEvent]
        );
    }
    
    // Track custom metrics
    async trackMetric(metricName, value, namespace = 'MyApp') {
        await this.cloudwatch.putMetricData(namespace, metricName, value);
    }
    
    // Monitor API performance
    async monitorAPIPerformance(endpoint, responseTime, statusCode) {
        await this.trackMetric('APIResponseTime', responseTime, 'MyApp/API');
        await this.trackMetric('APIStatusCode', statusCode, 'MyApp/API');
        
        if (responseTime > 5000) { // 5 seconds
            await this.logApplicationEvent('WARN', `Slow API response: ${endpoint}`, {
                responseTime,
                endpoint
            });
        }
    }
    
    // Monitor database performance
    async monitorDatabasePerformance(operation, duration, success) {
        await this.trackMetric('DatabaseOperationTime', duration, 'MyApp/Database');
        await this.trackMetric('DatabaseSuccess', success ? 1 : 0, 'MyApp/Database');
        
        if (!success) {
            await this.logApplicationEvent('ERROR', `Database operation failed: ${operation}`, {
                operation,
                duration
            });
        }
    }
}

// Complete Monitoring Setup
async function setupCompleteMonitoring() {
    const cloudwatchManager = new CloudWatchManager();
    const appMonitor = new ApplicationMonitor();
    
    try {
        // 1. Create log groups
        await cloudwatchManager.createLogGroup('/aws/lambda/myapp', 30);
        await cloudwatchManager.createLogGroup('/aws/ecs/myapp', 30);
        await cloudwatchManager.createLogGroup('/aws/apigateway/myapp', 30);
        
        // 2. Create log streams
        await cloudwatchManager.createLogStream('/aws/lambda/myapp', 'application-logs');
        await cloudwatchManager.createLogStream('/aws/ecs/myapp', 'container-logs');
        
        // 3. Create alarms
        await cloudwatchManager.createAlarm(
            'HighCPUUsage',
            'CPUUtilization',
            'AWS/EC2',
            80
        );
        
        await cloudwatchManager.createAlarm(
            'HighMemoryUsage',
            'MemoryUtilization',
            'AWS/EC2',
            85
        );
        
        await cloudwatchManager.createAlarm(
            'DatabaseConnections',
            'DatabaseConnections',
            'AWS/RDS',
            80
        );
        
        // 4. Create dashboard
        const dashboardWidgets = [
            {
                type: 'metric',
                x: 0,
                y: 0,
                width: 12,
                height: 6,
                properties: {
                    metrics: [
                        ['AWS/EC2', 'CPUUtilization'],
                        ['.', 'MemoryUtilization']
                    ],
                    period: 300,
                    stat: 'Average',
                    region: 'us-east-1',
                    title: 'EC2 Metrics'
                }
            },
            {
                type: 'log',
                x: 0,
                y: 6,
                width: 12,
                height: 6,
                properties: {
                    query: 'SOURCE \'/aws/lambda/myapp\' | fields @timestamp, @message | sort @timestamp desc | limit 100',
                    region: 'us-east-1',
                    title: 'Application Logs'
                }
            }
        ];
        
        await cloudwatchManager.createDashboard('MyApp-Dashboard', dashboardWidgets);
        
        // 5. Setup application monitoring
        await appMonitor.trackMetric('ApplicationStart', 1);
        await appMonitor.logApplicationEvent('INFO', 'Application monitoring setup completed');
        
        console.log('Complete monitoring setup finished');
    } catch (error) {
        console.error('Monitoring setup error:', error);
        throw error;
    }
}
```

---

## Cost Optimization

### Q12. How do you optimize AWS costs and implement cost management?
**Answer:** AWS cost optimization involves right-sizing resources, using appropriate pricing models, and implementing cost monitoring and governance.

**Key Strategies:**
- **Reserved Instances:** 1-3 year commitments for predictable workloads
- **Spot Instances:** Up to 90% savings for fault-tolerant workloads
- **S3 Lifecycle Policies:** Automatic data archiving
- **Auto Scaling:** Scale resources based on demand
- **Cost Allocation Tags:** Track spending by project/department
- **AWS Cost Explorer:** Analyze and forecast costs

**Real-time Example:**
```javascript
// AWS Cost Optimization and Management
const AWS = require('aws-sdk');
const costexplorer = new AWS.CostExplorer();
const ec2 = new AWS.EC2();

class CostOptimizationManager {
    constructor() {
        this.costexplorer = costexplorer;
        this.ec2 = ec2;
    }
    
    // Get cost and usage data
    async getCostAndUsage(startDate, endDate, granularity = 'MONTHLY') {
        const params = {
            TimePeriod: {
                Start: startDate,
                End: endDate
            },
            Granularity: granularity,
            Metrics: ['BlendedCost', 'UnblendedCost', 'UsageQuantity'],
            GroupBy: [
                {
                    Type: 'DIMENSION',
                    Key: 'SERVICE'
                }
            ]
        };
        
        try {
            const result = await this.costexplorer.getCostAndUsage(params).promise();
            return result.ResultsByTime;
        } catch (error) {
            console.error('Cost and usage query error:', error);
            throw error;
        }
    }
    
    // Get cost forecast
    async getCostForecast(startDate, endDate, metric = 'BLENDED_COST') {
        const params = {
            TimePeriod: {
                Start: startDate,
                End: endDate
            },
            Metric: metric,
            Granularity: 'MONTHLY',
            PredictionIntervalLevel: 95
        };
        
        try {
            const result = await this.costexplorer.getCostForecast(params).promise();
            return result.ForecastResultsByTime;
        } catch (error) {
            console.error('Cost forecast error:', error);
            throw error;
        }
    }
    
    // Get recommendations
    async getReservedInstancesRecommendations() {
        const params = {
            Service: 'EC2-Instance'
        };
        
        try {
            const result = await this.costexplorer.getReservationPurchaseRecommendation(params).promise();
            return result.Recommendations;
        } catch (error) {
            console.error('Reserved instances recommendations error:', error);
            throw error;
        }
    }
    
    // Analyze unused resources
    async analyzeUnusedResources() {
        const unusedResources = {
            unusedVolumes: [],
            unusedSnapshots: [],
            unusedElasticIPs: []
        };
        
        try {
            // Find unused EBS volumes
            const volumes = await this.ec2.describeVolumes({
                Filters: [
                    {
                        Name: 'status',
                        Values: ['available']
                    }
                ]
            }).promise();
            
            unusedResources.unusedVolumes = volumes.Volumes.map(vol => ({
                VolumeId: vol.VolumeId,
                Size: vol.Size,
                Type: vol.VolumeType,
                CreationDate: vol.CreateTime
            }));
            
            // Find old snapshots
            const snapshots = await this.ec2.describeSnapshots({
                OwnerIds: ['self'],
                Filters: [
                    {
                        Name: 'start-time',
                        Values: [`${new Date(Date.now() - 90 * 24 * 60 * 60 * 1000).toISOString().split('T')[0]}`]
                    }
                ]
            }).promise();
            
            unusedResources.unusedSnapshots = snapshots.Snapshots.map(snap => ({
                SnapshotId: snap.SnapshotId,
                VolumeSize: snap.VolumeSize,
                StartTime: snap.StartTime
            }));
            
            return unusedResources;
        } catch (error) {
            console.error('Unused resources analysis error:', error);
            throw error;
        }
    }
}

// Cost Optimization Strategies
class CostOptimizationStrategies {
    constructor() {
        this.costManager = new CostOptimizationManager();
    }
    
    // Implement S3 lifecycle policies
    async setupS3LifecyclePolicy(bucketName) {
        const s3 = new AWS.S3();
        
        const lifecycleConfiguration = {
            Rules: [
                {
                    ID: 'TransitionToIA',
                    Status: 'Enabled',
                    Transitions: [
                        {
                            Days: 30,
                            StorageClass: 'STANDARD_IA'
                        },
                        {
                            Days: 90,
                            StorageClass: 'GLACIER'
                        },
                        {
                            Days: 365,
                            StorageClass: 'DEEP_ARCHIVE'
                        }
                    ]
                },
                {
                    ID: 'DeleteOldVersions',
                    Status: 'Enabled',
                    NoncurrentVersionTransitions: [
                        {
                            NoncurrentDays: 30,
                            StorageClass: 'STANDARD_IA'
                        }
                    ],
                    NoncurrentVersionExpiration: {
                        NoncurrentDays: 90
                    }
                }
            ]
        };
        
        try {
            await s3.putBucketLifecycleConfiguration({
                Bucket: bucketName,
                LifecycleConfiguration: lifecycleConfiguration
            }).promise();
            
            console.log(`S3 lifecycle policy applied to ${bucketName}`);
        } catch (error) {
            console.error('S3 lifecycle policy error:', error);
            throw error;
        }
    }
    
    // Setup auto scaling for cost optimization
    async setupCostOptimizedAutoScaling(groupName, minSize, maxSize, targetCPU = 70) {
        const autoscaling = new AWS.AutoScaling();
        
        const params = {
            AutoScalingGroupName: groupName,
            MinSize: minSize,
            MaxSize: maxSize,
            DesiredCapacity: minSize,
            TargetGroupARNs: [process.env.TARGET_GROUP_ARN],
            HealthCheckType: 'ELB',
            HealthCheckGracePeriod: 300,
            Tags: [
                {
                    Key: 'CostOptimization',
                    Value: 'Enabled',
                    PropagateAtLaunch: true
                }
            ]
        };
        
        try {
            await autoscaling.updateAutoScalingGroup(params).promise();
            
            // Create scaling policies
            await autoscaling.putScalingPolicy({
                AutoScalingGroupName: groupName,
                PolicyName: 'ScaleUpPolicy',
                AdjustmentType: 'ChangeInCapacity',
                ScalingAdjustment: 1,
                Cooldown: 300
            }).promise();
            
            await autoscaling.putScalingPolicy({
                AutoScalingGroupName: groupName,
                PolicyName: 'ScaleDownPolicy',
                AdjustmentType: 'ChangeInCapacity',
                ScalingAdjustment: -1,
                Cooldown: 300
            }).promise();
            
            console.log(`Cost-optimized auto scaling configured for ${groupName}`);
        } catch (error) {
            console.error('Auto scaling configuration error:', error);
            throw error;
        }
    }
    
    // Implement cost allocation tags
    async setupCostAllocationTags() {
        const tagPolicies = [
            {
                TagKey: 'Environment',
                TagValues: ['Production', 'Staging', 'Development']
            },
            {
                TagKey: 'Project',
                TagValues: ['MyApp', 'DataPipeline', 'Analytics']
            },
            {
                TagKey: 'Department',
                TagValues: ['Engineering', 'Marketing', 'Sales']
            },
            {
                TagKey: 'CostCenter',
                TagValues: ['1001', '1002', '1003']
            }
        ];
        
        // Apply tags to resources
        const resources = await this.getResourcesForTagging();
        
        for (const resource of resources) {
            await this.applyCostAllocationTags(resource, {
                Environment: 'Production',
                Project: 'MyApp',
                Department: 'Engineering',
                CostCenter: '1001'
            });
        }
        
        console.log('Cost allocation tags applied');
    }
    
    // Generate cost optimization report
    async generateCostOptimizationReport() {
        const report = {
            timestamp: new Date().toISOString(),
            recommendations: [],
            currentCosts: {},
            potentialSavings: {}
        };
        
        try {
            // Get current costs
            const costData = await this.costManager.getCostAndUsage(
                new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString().split('T')[0],
                new Date().toISOString().split('T')[0]
            );
            
            report.currentCosts = costData;
            
            // Analyze unused resources
            const unusedResources = await this.costManager.analyzeUnusedResources();
            
            if (unusedResources.unusedVolumes.length > 0) {
                report.recommendations.push({
                    type: 'Unused EBS Volumes',
                    count: unusedResources.unusedVolumes.length,
                    potentialSavings: unusedResources.unusedVolumes.reduce((sum, vol) => sum + (vol.Size * 0.1), 0),
                    action: 'Delete unused volumes'
                });
            }
            
            // Get reserved instance recommendations
            const riRecommendations = await this.costManager.getReservedInstancesRecommendations();
            
            if (riRecommendations && riRecommendations.length > 0) {
                report.recommendations.push({
                    type: 'Reserved Instances',
                    recommendations: riRecommendations.length,
                    potentialSavings: riRecommendations.reduce((sum, rec) => sum + rec.EstimatedMonthlySavingsAmount, 0),
                    action: 'Purchase reserved instances'
                });
            }
            
            return report;
        } catch (error) {
            console.error('Cost optimization report error:', error);
            throw error;
        }
    }
}

// Complete Cost Optimization Setup
async function setupCompleteCostOptimization() {
    const costStrategies = new CostOptimizationStrategies();
    
    try {
        // 1. Setup S3 lifecycle policies
        await costStrategies.setupS3LifecyclePolicy('myapp-data-bucket');
        
        // 2. Setup cost-optimized auto scaling
        await costStrategies.setupCostOptimizedAutoScaling('MyApp-ASG', 2, 10);
        
        // 3. Setup cost allocation tags
        await costStrategies.setupCostAllocationTags();
        
        // 4. Generate optimization report
        const report = await costStrategies.generateCostOptimizationReport();
        
        console.log('Cost optimization setup completed');
        console.log('Optimization Report:', JSON.stringify(report, null, 2));
        
        return report;
    } catch (error) {
        console.error('Cost optimization setup error:', error);
        throw error;
    }
}
```

---

## Summary

This comprehensive AWS guide covers all essential topics for 7+ years experienced full-stack developers:

1. **AWS Basics & Core Services** - Understanding AWS ecosystem and core services
2. **EC2 & Compute Services** - Virtual server management and deployment
3. **S3 & Storage Services** - Object storage and file management
4. **RDS & Database Services** - Managed database solutions
5. **Lambda & Serverless** - Serverless computing and event-driven architecture
6. **VPC & Networking** - Network configuration and security
7. **IAM & Security** - Identity management and security best practices
8. **Load Balancing & Auto Scaling** - Traffic distribution and automatic scaling
9. **CloudFront & CDN** - Global content delivery
10. **Deployment & DevOps** - CI/CD pipelines and deployment strategies
11. **Monitoring & Logging** - Application monitoring and log management
12. **Cost Optimization** - Cost management and optimization strategies

Each section includes practical code examples, real-world scenarios, and best practices for production environments.
