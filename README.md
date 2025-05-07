# blog-app-MERN
MERN Stack Blog App Deployment on AWS
Scenario: Blog App Deployment
Deploy a MERN stack blog application on AWS using free-tier eligible services. The backend (Express) will run
on EC2, the frontend (React) will be hosted via S3 static website hosting, and MongoDB Atlas will be used as
the database. Media uploads will be handled through another S3 bucket with appropriate IAM permissions.

###   Part 1: S3 Bucket for Frontend

1.  Create an S3 bucket named `manal-blogapp-frontend` in the `eu-north-1` region.
2.  In the bucket's permissions, disable "Block all public access".
3.  In the bucket's properties, enable static website hosting.
4.  In the bucket's permissions, add the following bucket policy to allow public access to objects:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3::: manal-blogapp-frontend/*"
            }
        ]
    }
    ```
    ###   Part 2: S3 Bucket for Media Uploads

1.  Create a second S3 bucket named `manal-blogapp-media` in the `eu-north-1` region.
2.  In the bucket's permissions, disable "Block all public access".
3.  In the bucket's permissions, configure CORS (Cross-Origin Resource Sharing) to allow browser upload support:

    ```json
    [
        {
            "AllowedHeaders": ["*"],
            "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
            "AllowedOrigins": ["*"],
            "ExposeHeaders": ["ETag"]
        }
    ]
    ```
    ###   Part 3: IAM User and Policy for S3 Media Bucket Access
    1.  Create a new user with the username: `blog-app-user`
2.  In the "Permissions" section, select "Attach existing policies directly".
3.  Create a custom policy with the following JSON with your bucket name from Part 3 `manal-blogapp-media`:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:PutObject",
                    "s3:GetObject",
                    "s3:DeleteObject",
                    "s3:ListBucket"
                ],
                "Resource": [
                    "arn:aws:s3:::manal-blogapp-media",
                    "arn:aws:s3:::manal-blogapp-media/*"
                ]
            }
        ]
    }
    ```
    ###   Part 4: EC2 Backend Setup

1.  Launch a `t3.micro` instance in the `eu-north-1` region using an Ubuntu 22.04 LTS AMI.
2.  Configure the instance's security group to allow incoming traffic on the following ports:
    * SSH: 22 (for SSH access)
    * HTTP: 80
    * HTTPS: 443
    * Custom TCP: 5000
        * Inbound Rule for port 5000:
            * Type: Custom TCP
            * Protocol: TCP
            * Port Range: 5000
            * Source: 0.0.0.0/0
3.  Connect to the instance via SSH and run the following User Data script:

    ```bash
    #!/bin/bash
    apt update -y
    apt install -y git curl unzip tar gcc g++ make unzip

    su ubuntu << 'EOF'
    export NVM_DIR="$HOME/.nvm"
    curl -o- [https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh](https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh) | bash
    source "$NVM_DIR/nvm.sh"
    nvm install --lts
    nvm use --lts
    npm install -g pm2
    EOF

    # MongoDB Shell
    curl -L [https://downloads.mongodb.com/compass/mongosh-2.1.1-linux-x64.tgz](https://downloads.mongodb.com/compass/mongosh-2.1.1-linux-x64.tgz) -o mongosh.tgz
    tar -xvzf mongosh.tgz
    mv mongosh-*/bin/mongosh /usr/local/bin/mongosh
    chmod +x /usr/local/bin/mongosh
    rm -rf mongosh*

    # AWS CLI
    curl "[https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip](https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip)" -o "awscliv2.zip"
    unzip awscliv2.zip
    ./aws/install
    rm -rf aws awscliv2.zip
    ```
4.  Clone the MERN app repository
5.  Configure the backend, frontend environment variables
6.  Deploy the backend
7.  Deploy the backend
8.  Build and deploy the frontend
9.  
   ##   Cleanup

* Stop EC2 instance
* Remove IAM credentials
* Delete S3 buckets
