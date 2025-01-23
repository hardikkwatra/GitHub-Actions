# Task 1

```markdown
# Testing the PodState Script

Follow these steps to test the `PodState.py` script:

## 1. Install Docker

Make sure Docker is installed on your system. Follow these steps to install Docker:

### For Ubuntu:

1. **Update the apt package index and install packages to allow apt to use a repository over HTTPS:**
   ```sh
   sudo apt-get update
   sudo apt-get install \
       ca-certificates \
       curl \
       gnupg \
       lsb-release
   ```

2. **Add Docker's official GPG key:**
   ```sh
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

3. **Set up the stable repository:**
   ```sh
   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

4. **Install Docker Engine:**
   ```sh
   sudo apt-get update
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

5. **Verify that Docker Engine is installed correctly by running the hello-world image:**
   ```sh
   sudo docker run hello-world
   ```

## 2. Install Required Libraries

Make sure you have the necessary Python libraries installed:

```sh
pip install kubernetes
```

## 3. Create and Edit the Python Script

Create a new file and paste the script into it. Let's call the file `PodState.py`:

```sh
nano PodState.py
```
Paste your script into the file and save it.

## 4. Setup and Start Minikube (if not already running)

If you're using Minikube, start it:

```sh
minikube start
```

## 5. Ensure Kubeconfig is Correctly Set

Verify that your kubeconfig is correctly configured to point to your cluster:

```sh
kubectl config view
```

## 6. Check the Namespace

Ensure the namespace you are using (default is `default`) exists and has some pods. You can list all namespaces with:

```sh
kubectl get namespaces
```

## 7. List Pods in the Namespace

Check if there are any pods in the specified namespace:

```sh
kubectl get pods -n default
```

## 8. Scale Up/Down Your Deployment (Optional)

If you need to scale your deployment to generate pods:

```sh
kubectl scale deployment flask-app --replicas=1 -n default
```

or

```sh
kubectl scale deployment flask-app --replicas=0 -n default
```

## 9. Run the Script

Now you can run your Python script to check the pod statuses and fetch logs if necessary:

```sh
python3 PodState.py
```

## 10. Check the Log File

After running the script, you can check the `all-pods-logs.txt` file to verify the output:

```sh
cat all-pods-logs.txt
```


# Task 2

### Setting Up GitHub Actions Workflow

1. **Create the Workflow File**: Ensure you have created a workflow file in your repository at `.github/workflows/deploy.yml`.
2. **Configure Your Docker Hub Credentials**: 
   - Go to your repository settings on GitHub.
   - Navigate to `Settings` > `Secrets` and add `DOCKER_USERNAME` and `DOCKER_PASSWORD` with your Docker Hub credentials.
3. **Push Changes**: Push your code to the `main` branch. This will trigger the GitHub Actions workflow you have set up.

### Running the Linux Shell Script

1. **Create the Shell Script**: Ensure you have created a shell script named `setup.sh` in your project directory.
2. **Make the Script Executable**: 
   ```bash
   chmod +x setup.sh
   ```
3. **Execute the Script**:
   ```bash
   ./setup.sh
   ```

# Task 3

---

# Deploy Flask App on EKS and Monitor It

## Prerequisites
- AWS CLI configured with your credentials.
- Terraform installed.
- Docker installed.
- kubectl installed and configured.

## Step 1: Set Up EKS Cluster Using Terraform

### Create Terraform Configuration (main.tf)
```hcl
provider "aws" {
  region = "ap-south-1"
}

# Create IAM role for EKS Cluster
resource "aws_iam_role" "eks_cluster_role" {
  name = "eks-cluster-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect = "Allow",
        Principal = {
          Service = "eks.amazonaws.com"
        },
        Action = "sts:AssumeRole"
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "eks_cluster_policy_attachment" {
  role       = aws_iam_role.eks_cluster_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
}

# Create IAM role for EKS Node Group
resource "aws_iam_role" "eks_node_role" {
  name = "eks-node-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect = "Allow",
        Principal = {
          Service = "ec2.amazonaws.com"
        },
        Action = "sts:AssumeRole"
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "eks_worker_node_policy_attachment" {
  role       = aws_iam_role.eks_node_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
}

resource "aws_iam_role_policy_attachment" "eks_cni_policy_attachment" {
  role       = aws_iam_role.eks_node_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
}

resource "aws_iam_role_policy_attachment" "eks_ecr_read_only_policy_attachment" {
  role       = aws_iam_role.eks_node_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
}

resource "aws_iam_role_policy_attachment" "ssm_managed_policy_attachment" {
  role       = aws_iam_role.eks_node_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
}

# Create VPC
resource "aws_vpc" "my_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
}

# Create Subnets with Auto-Assign Public IP
resource "aws_subnet" "my_subnet_1" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-south-1a"
  map_public_ip_on_launch = true
}

resource "aws_subnet" "my_subnet_2" {
  vpc_id                  = aws_vpc.my_vpc.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "ap-south-1b"
  map_public_ip_on_launch = true
}

# Create Internet Gateway
resource "aws_internet_gateway" "my_igw" {
  vpc_id = aws_vpc.my_vpc.id
}

# Create Route Table and Route
resource "aws_route_table" "my_route_table" {
  vpc_id = aws_vpc.my_vpc.id
}

resource "aws_route" "my_route" {
  route_table_id         = aws_route_table.my_route_table.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.my_igw.id
}

# Associate Route Table with Subnets
resource "aws_route_table_association" "a" {
  subnet_id      = aws_subnet.my_subnet_1.id
  route_table_id = aws_route_table.my_route_table.id
}

resource "aws_route_table_association" "b" {
  subnet_id      = aws_subnet.my_subnet_2.id
  route_table_id = aws_route_table.my_route_table.id
}

# Create Security Group
resource "aws_security_group" "eks_security_group" {
  vpc_id = aws_vpc.my_vpc.id

  ingress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Create EKS Cluster
resource "aws_eks_cluster" "my_eks_cluster" {
  name     = "my-eks-cluster"
  role_arn = aws_iam_role.eks_cluster_role.arn

  vpc_config {
    subnet_ids         = [aws_subnet.my_subnet_1.id, aws_subnet.my_subnet_2.id]
    security_group_ids = [aws_security_group.eks_security_group.id]
  }
}

# Create EKS Node Group
resource "aws_eks_node_group" "my_node_group" {
  cluster_name    = aws_eks_cluster.my_eks_cluster.name
  node_role_arn   = aws_iam_role.eks_node_role.arn
  subnet_ids      = [aws_subnet.my_subnet_1.id, aws_subnet.my_subnet_2.id]
  instance_types  = ["t2.micro"]

  scaling_config {
    desired_size = 2
    min_size     = 1
    max_size     = 3
  }

  remote_access {
    ec2_ssh_key = "MyKey"
  }

  tags = {
    Name = "my-eks-node-group"
  }
}
```
2. **Apply Terraform Configuration**
    ```bash
    terraform apply
    ```

## Step 2: Build and Push Docker Image

1. **Create Dockerfile for Flask App**
    ```Dockerfile
    FROM python:3.9-slim
    WORKDIR /app
    COPY . /app
    RUN pip install flask
    EXPOSE 5000
    CMD ["flask", "run", "--host=0.0.0.0", "--port=5000"]
    ```
2. **Build Docker Image**
    ```bash
    docker build -t my-flask-app .
    ```
3. **Push Docker Image to Docker Hub**
    ```bash
    docker login
    docker push hardddyy/flask-app:latest
    ```
## Step 3: Update kubeconfig and Apply Kubernetes Manifests

1. **Update kubeconfig to Use EKS Cluster**
    ```bash
    aws eks --region ap-south-1 update-kubeconfig --name my-eks-cluster
    ```

2. **Create Kubernetes Deployment and Service Manifests**

    **deployment.yaml**
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: flask-app
      labels:
        app: flask-app
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: flask-app
      template:
        metadata:
          labels:
            app: flask-app
        spec:
          containers:
          - name: flask-app
            image: hardddyy/flask-app:latest
            ports:
            - containerPort: 5000
    ```

    **service.yaml**
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: flask-app-service
    spec:
      selector:
        app: flask-app
      type: NodePort
      ports:
        - protocol: TCP
          port: 5000
          targetPort: 5000
          nodePort: 32188
    ```

3. **Apply Kubernetes Deployment and Service**
    ```bash
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    ```

## Step 4: Verify Deployment

1. **Check Deployments, Pods, and Services**
    ```bash
    kubectl get deployments
    kubectl get pods
    kubectl get services
    kubectl get nodes -o wide
    ```

2. **Access Flask App via Public Endpoint**
    ```bash
    curl http://<public-endpoint-ip>:<port>
    ```

## Step 5: Set Up Monitoring Using CloudWatch

### Create IAM Policies and Roles for CloudWatch
1. **Create IAM Policy for CloudWatch Agent**
    - Go to the [IAM console](https://console.aws.amazon.com/iam/).
    - Click on "Policies" and then "Create policy".
    - Use the following JSON policy:
    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "logs:CreateLogGroup",
            "logs:CreateLogStream",
            "logs:PutLogEvents",
            "logs:DescribeLogStreams",
            "cloudwatch:PutMetricData"
          ],
          "Resource": "*"
        }
      ]
    }
    ```
    - Click "Review policy", give it a name (e.g., `CloudWatchAgentPolicy`), and then create the policy.

2. **Create IAM Role for EC2 Instances**
    - Go to the IAM console.
    - Click on "Roles" and then "Create role".
    - Select "AWS service" and then "EC2".
    - Attach the `CloudWatchAgentPolicy` you just created.
    - Name the role (e.g., `EC2CloudWatchAgentRole`) and create the role.

### Configure CloudWatch Agent on EC2 Instances
1. **Install CloudWatch Agent on EC2 Instances**
    - Connect to your EC2 instance via SSH.
    - Run the following commands to install the CloudWatch agent:
    ```bash
    sudo yum update -y
    sudo yum install -y amazon-cloudwatch-agent
    ```

2. **Create CloudWatch Agent Configuration File**
    - Create the configuration file (`/opt/aws/amazon-cloudwatch-agent/bin/config.json`):
    ```json
    {
      "agent": {
        "metrics_collection_interval": 60,
        "logfile": "/var/log/amazon-cloudwatch-agent.log",
        "run_as_user": "root"
      },
      "logs": {
        "logs_collected": {
          "files": {
            "collect_list": [
              {
                "file_path": "/var/log/messages",
                "log_group_name": "my-flask-app-log-group",
                "log_stream_name": "{instance_id}-messages",
                "timestamp_format": "%b %d %H:%M:%S"
              }
            ]
          }
        }
      },
      "metrics": {
        "append_dimensions": {
          "InstanceId": "${aws:InstanceId}"
        },
        "metrics_collected": {
          "cpu": {
            "measurement": [
              "cpu_usage_idle",
              "cpu_usage_iowait",
              "cpu_usage_user",
              "cpu_usage_system"
            ],
            "metrics_collection_interval": 60,
            "resources": [
              "*"
            ]
          },
          "disk": {
            "measurement": [
              "used_percent"
            ],
            "metrics_collection_interval": 60,
            "resources": [
              "*"
            ]
          },
          "mem": {
            "measurement": [
              "mem_used_percent"
            ],
            "metrics_collection_interval": 60
          }
        }
      }
    }
    ```

3. **Start the CloudWatch Agent**
    - Run the following command to start the CloudWatch agent with the configuration file:
    ```bash
    sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
    ```

### Configure CloudWatch Agent on EKS Nodes
1. **Install CloudWatch Agent on EKS Nodes**
    - Create a DaemonSet to deploy the CloudWatch agent on all nodes in your EKS cluster. Create a file named `cloudwatch-agent-daemonset.yaml`:
    ```yaml
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: cloudwatch-agent
      namespace: amazon-cloudwatch
      labels:
        k8s-app: cloudwatch-agent
    spec:
      selector:
        matchLabels:
          name: cloudwatch-agent
      template:
        metadata:
          labels:
            name: cloudwatch-agent
        spec:
          serviceAccountName: cloudwatch-agent
          containers:
            - name: cloudwatch-agent
              image: amazon/cloudwatch-agent:latest
              resources:
                limits:
                  memory: 200Mi
                  cpu: 200m
                requests:
                  memory: 200Mi
                  cpu: 100m
              volumeMounts:
                - name: config-volume
                  mountPath: /etc/cwagentconfig
          volumes:
            - name: config-volume
              configMap:
                name: cloudwatch-agent-config
    ```

2. **Create the Configuration ConfigMap**
    - Create a file named `cloudwatch-agent-configmap.yaml` with the following content:
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: cloudwatch-agent-config
      namespace: amazon-cloudwatch
      labels:
        k8s-app: cloudwatch-agent
    data:
      config.json: |
        {
          "agent": {
            "metrics_collection_interval": 60,
            "logfile": "/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log",
            "run_as_user": "cwagent"
          },
          "logs": {
            "logs_collected": {
              "files": {
                "collect_list": [
                  {
                    "file_path": "/var/log/containers/*.log",
                    "log_group_name": "/aws/containerinsights/cluster-name/logs",
                    "log_stream_name": "k8s-logs/{node_name}/container-name/$(k8s_namespace_name)/$(k8s_pod_name)",
                    "timezone": "UTC"
                  }
                ]
              }
            }
          },
          "metrics": {
            "append_dimensions": {
              "ClusterName": "cluster-name",
              "NodeName": "${aws:InstanceId}"
            },
            "metrics_collected": {
              "cpu": {
                "measurement": [
                  "cpu_usage_active",
                  "cpu_usage_idle",
                  "cpu_usage_iowait",
                  "cpu_usage_user",
                  "cpu_usage_system"
                ],
                "metrics_collection_interval": 60
              },
              "disk": {
                "measurement": [
                  "used_percent"
                ],
                "metrics_collection_interval": 60,
                "resources": [
                  "*"
                ]
              },
              "mem": {
                "measurement": [
                  "mem_used_percent"
                ],
                "metrics_collection_interval": 60
              }
            }
          }
        }
    ```

3. **Deploy the DaemonSet and ConfigMap**
    ```bash
    kubectl create namespace amazon-cloudwatch
    kubectl apply -f cloudwatch-agent-configmap.yaml
    kubectl apply -f cloudwatch-agent-daemonset.yaml
    ```

### Create Dashboards in CloudWatch
1. **Open CloudWatch Console**
    - Go to the [AWS Management Console](https://console.aws.amazon.com/cloudwatch/).
    - Navigate to CloudWatch.

2. **Create a New Dashboard**
    - Click on "Dashboards" > "Create dashboard".
    - Name your dashboard and select a layout.

3. **Add Widgets for Metrics**
    - **EC2 Instances Metrics**:
        - Click on "Add widget" > "Line".
        - Select metrics like `CPUUtilization`, `MemoryUtilization`, and other relevant instance metrics.
    - **EKS Cluster Metrics**:
        - Add widgets for scheduler metrics (`scheduler_schedule_attempts_*`, `scheduler_pending_pods_*`).
        - Add widgets for API server metrics (`apiserver_request_total_*`, `apiserver_request_duration_seconds_*`, `apiserver_current_inflight_requests_*`).
        - Add widgets for storage metrics (`apiserver_storage_size_bytes`).

4. **Save and View Dashboard**
    - Save the dashboard.
    - Access the dashboard to monitor the metrics.

---

