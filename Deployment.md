This is the **Evaluator-X High-Level Development & Deployment Guide**. It is designed for a "team of 3" with zero prior AWS knowledge.

This guide bypasses all commercial layers (Stripe, API Gateway) and focuses purely on deploying a production-grade, scalable **Code Execution Engine**.

-----

### **PHASE 0: The Artifacts (Local Preparation)**

Before touching AWS, you must create a "deployment package" locally.

**1. The "Golden Script" (`build_ami.sh`)**
You need a single script that turns a fresh Ubuntu server into an Evaluator-X worker. Save this script locally. It must do the following:

  * **Install Dependencies:** `apt-get install -y build-essential libcap-dev ...`
  * **Install Isolate:** Clone `isolate`, `make`, `make install`.
  * **Create Chroots:** Run your massive chroot generation logic (`/opt/evalx/chroots/...`).
  * **Install CloudWatch Agent:** Download the `.deb` file from Amazon and install it (crucial for metrics).
  * **Setup Systemd:** Create a service file (`/etc/systemd/system/evalx.service`) so your app restarts if it crashes.
      * *Tip:* Leave the `ExecStart` command generic or use a placeholder variable.

**2. The Rust Binary**

  * Compile your project locally with `cargo build --release`.
  * You will upload this binary to your servers later (or compile it inside the Golden Script if you want to keep it simple).

-----

### **PHASE 1: The Virtual Datacenter (Network Layer)**

Think of this as buying an empty building and installing cables.

**1. Create a VPC (Virtual Private Cloud)**

  * **Name:** `evalx-vpc`
  * **IPv4 CIDR:** `10.0.0.0/16` (This gives you 65,000 internal IP addresses).

**2. Create Subnets (The Rooms)**
You need three "rooms" in your VPC.

  * **Public Subnet (`10.0.1.0/24`):** For the Load Balancer. It has a door to the outside world.
  * **Private App Subnet (`10.0.2.0/24`):** For your API & Workers. No direct internet access (secure).
  * **Private Data Subnet (`10.0.3.0/24`):** For Redis. Deeply secure.

**3. The Internet Gateways**

  * **Internet Gateway (IGW):** Attach this to your VPC. This is the front door.
  * **NAT Gateway:** Create this in the **Public Subnet**.
      * *Why:* Your Private servers (Workers) need to talk to AWS CloudWatch (to send metrics), but they shouldn't accept incoming connection requests from the internet. They will route their outgoing traffic through this NAT Gateway.

**4. Route Tables (The Map)**

  * **Public Route Table:** Point `0.0.0.0/0` -\> `Internet Gateway`. Associate with **Public Subnet**.
  * **Private Route Table:** Point `0.0.0.0/0` -\> `NAT Gateway`. Associate with **Private App Subnet**.

-----

### **PHASE 2: The Memory (Redis Layer)**

This is the "Brain" that connects your API and Workers.

**1. Create a Security Group (`redis-sg`)**

  * **Inbound Rule:** Allow TCP port `6379` *only* from the `10.0.0.0/16` (Your VPC CIDR).
  * *Why:* No one outside your virtual building can even try to connect to Redis.

**2. Create AWS ElastiCache (Redis)**

  * **Engine:** Redis OSS.
  * **Node Type:** `cache.t3.micro` (Cheap, good for dev) or `cache.m5.large` (Production).
  * **Subnet Group:** Choose your **Private Data Subnet**.
  * **Security Group:** Attach `redis-sg`.
  * **Result:** AWS gives you a "Primary Endpoint" (e.g., `master.evalx.cache.aws...`). **Copy this.**

-----

### **PHASE 3: The Golden AMI (The Hard Drive)**

Now we build the "Standard Issue" server image.

**1. Launch a Temporary EC2 Instance**

  * **OS:** Ubuntu 24.04 LTS.
  * **Subnet:** **Public Subnet** (so you can SSH into it easily for this setup phase).
  * **Security Group:** Allow SSH (port 22) from *My IP*.

**2. Provision the Server**

  * SSH into the instance.
  * Upload your Rust binary (`evalx`) to `/usr/local/bin/`.
  * Upload your `config/` folder to `/etc/evalx/`.
  * **Run your `build_ami.sh`** (Install isolate, create chroots, install CW Agent).
  * **Configure CloudWatch Agent:** Create a file `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json`:
    ```json
    {
      "metrics": {
        "metrics_collected": {
          "prometheus": {
            "prometheus_config_path": "/etc/evalx/prometheus.yaml",
            "emf_processor": {
              "metric_declaration": [
                {
                  "source_labels": ["job"],
                  "label_matcher": "^evalx$",
                  "dimensions": [["language"]],
                  "metric_selectors": [
                    "^evalx_queue_depth_total$"
                  ]
                }
              ]
            }
          }
        }
      }
    }
    ```
    *This tells AWS: "Read metrics from my app and look for `evalx_queue_depth_total`."*

**3. Bake the Image**

  * In AWS Console, select the instance -\> **Actions** -\> **Image** -\> **Create Image**.
  * **Name:** `EvalX-Golden-v1`.
  * *Wait 10 minutes. You now have a cloned hard drive ready to be multiplied.*
  * (Terminate the temporary instance).

-----

### **PHASE 4: The Orchestration (Launch Templates)**

We create two "Startup Plans" using the **same** Golden AMI.

**1. The IAM Role (The Badge)**

  * Create a Role named `EvalX-Instance-Role`.
  * Attach Policy: `CloudWatchAgentServerPolicy`.
  * *Why:* Gives your servers permission to write metrics to CloudWatch.

**2. Launch Template A: "API-Server"**

  * **AMI:** `EvalX-Golden-v1`.
  * **Instance Type:** `t3.small`.
  * **IAM Role:** `EvalX-Instance-Role`.
  * **Security Group:** Allow TCP `3000` from "Everywhere" (We will lock this down to Load Balancer later).
  * **User Data (Advanced Details):**
    ```bash
    #!/bin/bash
    export REDIS_HOST=master.evalx.cache.aws...
    systemctl start amazon-cloudwatch-agent
    /usr/local/bin/evalx  # Starts in SERVER mode by default
    ```

**3. Launch Template B: "Worker-Node"**

  * **AMI:** `EvalX-Golden-v1`.
  * **Instance Type:** `c5.large` (Compute optimized for compilation).
  * **IAM Role:** `EvalX-Instance-Role`.
  * **Security Group:** Allow NO inbound traffic (Workers initiate connections out to Redis).
  * **User Data:**
    ```bash
    #!/bin/bash
    export REDIS_HOST=master.evalx.cache.aws...
    systemctl start amazon-cloudwatch-agent
    /usr/local/bin/evalx worker # Starts in WORKER mode
    ```

-----

### **PHASE 5: The Scaling Groups (ASG)**

**1. Create Auto Scaling Group: `EvalX-API-ASG`**

  * **Launch Template:** `API-Server`.
  * **VPC/Subnets:** **Private App Subnets**.
  * **Scaling Policy:** "Target Tracking" -\> Average CPU Utilization = 60%.
  * *Result:* If CPU gets hot, AWS adds more API servers.

**2. Create Auto Scaling Group: `EvalX-Worker-ASG`**

  * **Launch Template:** `Worker-Node`.
  * **VPC/Subnets:** **Private App Subnets**.
  * **Scaling Policy:** **Step Scaling** (This is the trick).
      * Go to CloudWatch -\> Alarms -\> Create Alarm.
      * Metric: `CWAgent > evalx_queue_depth_total`.
      * Condition: `> 20`.
      * Action: "Add 2 instances".
  * *Result:* If Redis queue fills up, AWS adds more Workers.

-----

### **PHASE 6: Exposure (The Load Balancer)**

Your API servers are hiding in a private subnet. We need a public bridge.

**1. Create Target Group**

  * **Type:** Instances.
  * **Port:** 3000.
  * **VPC:** `evalx-vpc`.
  * **Register Targets:** Select your `EvalX-API-ASG`.

**2. Create Application Load Balancer (ALB)**

  * **Scheme:** Internet-facing.
  * **Subnets:** **Public Subnets**.
  * **Listener:** HTTP Port 80 (forward to Target Group).
  * *Result:* AWS gives you a DNS name (e.g., `evalx-lb-12345.aws.com`).

-----

### **VERIFICATION & FUTURE**

**How to test:**

1.  Hit `http://evalx-lb-12345.aws.com/execute/batch`.
2.  The LB forwards to an API Server (Private Subnet).
3.  API Server pushes job to Redis (Private Subnet).
4.  API Server returns "Queued".
5.  Worker (Private Subnet) wakes up, pulls job, isolates, runs, updates Redis.
6.  Your polling script gets "Completed".

**Can this be an API Service?**
Yes.
This entire architecture is "The Engine".
To sell it, you simply place **AWS API Gateway** in front of your Load Balancer.

  * **Today:** `User -> Load Balancer -> API`
  * **Tomorrow:** `User -> API Gateway (Auth/Billing) -> Load Balancer -> API`

This setup is professional, secure, and scales infinitely.

[Running services with User Data](https://www.google.com/search?q=https://www.youtube.com/watch%3Fv%3D-dYXW0Rh7b8)

This video is relevant because it visually demonstrates how the "User Data" script works on EC2 startup, which is the critical mechanism you will use to distinguish your API servers from your Worker servers using the same AMI.
