## 🚀 Full Deployment Guide: `evalx-full-system` to AWS EC2

This guide documents the end-to-end process, from uploading your local Docker image to AWS ECR to launching and verifying the full application stack on a new EC2 server.

-----

### **Phase 1: Push Local Image to AWS ECR (The "Golden Image")**

This phase moves your 2.76GB image from your laptop to a private, secure registry in the cloud.

#### **Step 1. Verify Local Image**

  * **Where:** Your Local Laptop Terminal (`gunasekhar@...`)
  * **Action:** Confirm the image name and tag.
  * **Command:**
    ```bash
    docker images | grep evalx-full-system
    ```
  * **Result:** `evalx-full-system   latest   27b6a3526184   ...`

#### **Step 2. Create ECR Repository**

  * **Where:** AWS Management Console (Web Browser)
  * **Action:** Create a new repository to hold your image.
  * **Details:**
    1.  Navigate to the **Elastic Container Registry (ECR)** service.
    2.  Ensure you are in the correct region: **Asia Pacific (Mumbai) (ap-south-2)**.
    3.  Click **"Create repository"**.
    4.  Set **Visibility** to **Private**.
    5.  Set **Repository name** to `evalx-full-system`.
    6.  Click **"Create repository"**.

#### **Step 3. Authenticate Local Terminal to ECR**

  * **Where:** Your Local Laptop Terminal (`gunasekhar@...`)
  * **Action:** Log your local Docker client into the AWS ECR service.
  * **Command:**
    ```bash
    aws ecr get-login-password --region ap-south-2 | docker login --username AWS --password-stdin 991246619733.dkr.ecr.ap-south-2.amazonaws.com
    ```
  * **Result:** `Login Succeeded`

#### **Step 4. Tag Local Image for ECR**

  * **Where:** Your Local Laptop Terminal (`gunasekhar@...`)
  * **Action:** Apply an "address label" to your local image, pointing it to the ECR repository.
  * **Command:**
    ```bash
    docker tag evalx-full-system:latest 991246619733.dkr.ecr.ap-south-2.amazonaws.com/evalx-full-system:latest
    ```
  * **Result:** No output.

#### **Step 5. Push Image to ECR**

  * **Where:** Your Local Laptop Terminal (`gunasekhar@...`)
  * **Action:** Upload the 2.76GB image to the repository.
  * **Command:**
    ```bash
    docker push 991246619733.dkr.ecr.ap-south-2.amazonaws.com/evalx-full-system:latest
    ```
  * **Result:** A series of progress bars, ending with a `sha256` digest.

-----

### **Phase 2: Launch and Connect to Production Server (EC2)**

With the image safe in ECR, you now build the server that will run it.

#### **Step 6. Launch EC2 Instance**

  * **Where:** AWS Management Console (Web Browser)
  * **Action:** Create a new virtual server.
  * **Details:**
    1.  Navigate to the **EC2** service.
    2.  Click **"Launch instances"**.
    3.  **Name:** `evalx-production-ccppjavapython`
    4.  **AMI:** `Ubuntu 22.04 LTS`
    5.  **Instance Type:** `t3.medium` (or `t2.medium`)
    6.  **Key Pair:** `evalx-key-jccp` (and download the `.pem` file).
    7.  **Network Settings:** Click **"Edit"**.
          * Create a new security group named `evalx-sg`.
          * **Inbound Rule 1 (SSH):** Type `SSH`, Source `Anywhere (0.0.0.0/0)` (or `My IP`).
          * **Inbound Rule 2 (API):** Type `Custom TCP`, Port `3000`, Source `Anywhere (0.0.0.0/0)`.
    8.  **Advanced details (CRITICAL):**
          * Expand **"Advanced details"**.
          * For **"IAM instance profile"**, select the role with `AmazonEC2ContainerRegistryReadOnly` policy.

#### **Step 7. Connect to Instance via SSH**

  * **Where:** Your Local Laptop Terminal (`gunasekhar@...`)
  * **Action:** Log in to the new server's terminal.
  * **Details:**
    1.  **Set Key Permissions** (one-time command):
        ```bash
        chmod 400 ~/Downloads/evalx-key-jccp.pem
        ```
    2.  **Log In** (using your instance's Public IP):
        ```bash
        ssh -i ~/Downloads/evalx-key-jccp.pem ubuntu@13.232.19.131
        ```
  * **Result:** Your prompt changes to `ubuntu@ip-172-31-37-112:~$`.

-----

### **Phase 3: Deploy the Application Stack on EC2**

You are now inside the new server. All commands are run here.

#### **Step 8. Install Software on Server**

  * **Where:** EC2 Server Terminal (`ubuntu@ip-...`)
  * **Action:** Install Docker and the AWS CLI.
  * **Command:**
    ```bash
    sudo apt-get update
    sudo apt-get install -y docker.io awscli
    sudo systemctl start docker
    sudo systemctl enable docker
    ```

#### **Step 9. Authenticate EC2 Server to ECR**

  * **Where:** EC2 Server Terminal (`ubuntu@ip-...`)
  * **Action:** Grant the server's Docker permission to pull from ECR (uses the IAM Role).
  * **Command:**
    ```bash
    aws ecr get-login-password --region ap-south-2 | sudo docker login --username AWS --password-stdin 991246619733.dkr.ecr.ap-south-2.amazonaws.com
    ```
  * **Result:** `Login Succeeded`

#### **Step 10. Pull the Image to the Server**

  * **Where:** EC2 Server Terminal (`ubuntu@ip-...`)
  * **Action:** Download the 2.76GB image from ECR to the server.
  * **Command:**
    ```bash
    sudo docker pull 991246619733.dkr.ecr.ap-south-2.amazonaws.com/evalx-full-system:latest
    ```
  * **Result:** `Pull complete`.

#### **Step 11. Create the Docker Network**

  * **Where:** EC2 Server Terminal (`ubuntu@ip-...`)
  * **Action:** Create a private network for the containers.
  * **Command:**
    ```bash
    sudo docker network create evalx-net
    ```

#### **Step 12. Launch the Redis Container**

  * **Where:** EC2 Server Terminal (`ubuntu@ip-...`)
  * **Action:** Start the Redis database/queue.
  * **Command:**
    ```bash
    sudo docker run -d \
        --name redis \
        --network evalx-net \
        --restart always \
        redis:7.0-alpine
    ```

#### **Step 13. Launch the API Server Container**

  * **Where:** EC2 Server Terminal (`ubuntu@ip-...`)
  * **Action:** Start the API server, expose port 3000, and connect it to Redis. We use `--entrypoint` to bypass the worker-only setup script.
  * **Command:**
    ```bash
    sudo docker run -d \
        --name evalx-server \
        --network evalx-net \
        -p 3000:3000 \
        -e REDIS_HOST=redis \
        --restart always \
        --entrypoint /app/evalx \
        991246619733.dkr.ecr.ap-south-2.amazonaws.com/evalx-full-system:latest \
        server
    ```

#### **Step 14. Launch the Worker Container**

  * **Where:** EC2 Server Terminal (`ubuntu@ip-...`)
  * **Action:** Start the worker with privileged access to create sandboxes. We provide the full command `/app/evalx worker`.
  * **Command:**
    ```bash
    sudo docker run -d \
        --name evalx-worker \
        --network evalx-net \
        --privileged \
        --cgroupns=host \
        -v /sys/fs/cgroup:/sys/fs/cgroup:rw \
        -e REDIS_HOST=redis \
        --restart always \
        991246619733.dkr.ecr.ap-south-2.amazonaws.com/evalx-full-system:latest \
        /app/evalx worker
    ```

-----

### **Phase 4: Final System Verification**

#### **Step 15. Check Container Status**

  * **Where:** EC2 Server Terminal (`ubuntu@ip-...`)
  * **Action:** Verify all three containers are running and not restarting.
  * **Command:**
    ```bash
    sudo docker ps
    ```
  * **Result:** `evalx-worker`, `evalx-server`, and `redis` must all show `STATUS` as "Up".

#### **Step 16. Internal Test (Localhost)**

  * **Where:** EC2 Server Terminal (`ubuntu@ip-...`)
  * **Action:** Test that the server and worker can communicate inside the server.
  * **Command:**
    ```bash
    curl -X POST http://localhost:3000/execute \
    -H "Content-Type: application/json" \
    -d '{
        "language": "python",
        "version": "3.9",
        "code": "print(\"Hello from the full system!\")"
    }'
    ```
  * **Result:** `{"token":"...","status":"Queued",...}`. You can then poll this token to see `"status":"Completed"`.

#### **Step 17. Public Test (Production)**

  * **Where:** Your Local Laptop Terminal (`gunasekhar@...`)
  * **Action:** Test the full system from the public internet.
  * **Command 1 (Submit):**
    ```bash
    curl -X POST http://13.232.19.131:3000/execute \
    -H "Content-Type: application/json" \
    -d '{
        "language": "cpp",
        "version": "11",
        "code": "#include <iostream>\nint main() { std::cout << \"Hello from the Public Internet!\" << std::endl; return 0; }",
        "stdin": "",
        "timeout": 5
    }'
    ```
  * **Result 1:** `{"token":"197c6ba6-ed88-4500-9ce1-0cf3f8e97aef",...}`
  * **Command 2 (Get Result):**
    ```bash
    curl http://13.232.19.131:3000/submissions/197c6ba6-ed88-4500-9ce1-0cf3f8e97aef
    ```
  * **Result 2:** `{"token":"...","status":"Completed","results":[{"stdout":"Hello from the Public Internet!\n",...}]}`
