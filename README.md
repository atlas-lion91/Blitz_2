# Blitz_2:  Monitoring Server and Application Performance



---

*Date:* October 9, 2023

*Authored by:* Khalil Elkharbibi

## Objective

Following the rollout of our updated URL Shortener application, a large-scale test involving 14,000 server requests was conducted. Unfortunately 500 of these were unsuccessful. We must devise a strategy to handle all 14,000 server requests successfully.

## Testing Setup

1. **Application Logging Enhancement**

  - Verified AWS CLI Installation Locally:
  - Checked for the presence of AWS CLI locally and confirmed it was not installed.
  - Installed AWS CLI using the command: `sudo apt install awscli`.
  - Verified the successful installation by running: `aws --version`.

- Configured AWS CLI:
  - Configured AWS CLI using the command: `aws configure`.

- Ensured Application Availability:
  - Confirmed that the application was running.

- Added Logging to the Application:
  - Imported the logging module.
  - Configured logging to write to the 'app.log' file using `logging.basicConfig`.
    
  ![image](https://github.com/atlas-lion91/Blitz_2/assets/140761974/dac40630-e388-4f0a-b4b0-823c99d844f2)

2. **Nginx Configuration Updates:**

- Configured Nginx to efficiently handle a higher number of requests.
- Updated the setting "worker_processes" from "auto" to "8" to optimize server performance.
- Increased the "worker_connections" from "768" to "2000" to accommodate more concurrent connections.
- Uncommented the line "multi_accept on;" to enable the "multi_accept" feature for improved efficiency.

   ![Nginx_config](https://github.com/atlas-lion91/Blitz_2/assets/140761974/8a26c906-3dad-47a6-9a9b-fef67759e6d6)
- Optimized Nginx for quicker loading times by enabling Gzip compression.
  <img width="1276" alt="Nginx config 2" src="https://github.com/atlas-lion91/Blitz_2/assets/140761974/c07bb840-1ce8-42ff-b1ca-9ee64a707b37">

 *Nginx serves as a proxy server.*
 *Gunicorn is responsible for running the Flask application.*

*Running Nginx in front of the application serves the following purposes:*

- *Protection:* It acts as a protective layer for the application.

- *Data Persistence:* While the application currently saves data in a JSON file, future plans include implementing more robust data persistence solutions.

- *Security Enhancement:* An additional layer of security is achieved by closing the Gunicorn port and routing incoming traffic through Nginx.




3. **Stress Test Package Installation**

   ```bash
   sudo apt install stress-ng
   ```

4. **Stress Test Script**

   Used the script below to intensively test two CPU cores at a higher priority.

   ```bash
   sudo nice -n -20 stress-ng --cpu 2
   ```
   - This script runs the stress-ng program with a high-priority setting, simulating a CPU-intensive workload using two CPU cores, and it runs this process in the background. This can be used for testing how well a system handles high CPU load or for benchmarking purposes.

## Performance Outcomes

Despite our preparations, the QA test was not entirely successful. We found that 300 of the 14,000 requests failed. Additionally, our CPU utilization skyrocketed to 100%.

![QA Test Outcome](images/QA_Test_Notification.png)

### Reliability Testing and Insights

Running a heavy load of 14,000 requests while concurrently executing the `stress-ng` script stretched our server's capabilities. This test was designed to gauge how well our server could cope under extreme conditions.

#### Testing on t2.medium

Examples of CPU utilization on a t2.medium instance (2 CPUs) when subjected to stress:

- ![Stress Test Result 1](images/Deploy_4_user1_Stress_Test.png)
- ![Stress Test Result 2](images/Deploy_4_user0_Stress_Test.png)

Our observation was that the two CPUs were nearly saturated at 99% even without any active requests or Jenkins builds. We believe we might need to upgrade our resources, given the three-tier structure of our application.

#### Testing on t2.xlarge

The same stress test on a t2.xlarge instance produced the following outcomes:

- ![Stress Test Result 1](images/CPU_1_Deploy.png)
- ![Stress Test Result 2](images/CPU_2_Deploy.png)


Considering our server's multi-tier architecture, encompassing the Web, Application, and Data tiers, simply doubling the CPUs might not suffice.

---

## Implementation Steps

### 1. Infrastructure Blueprint 

   ![Blitz 2 drawio](https://github.com/atlas-lion91/Blitz_2/assets/140761974/35a61ce2-c246-4f27-8f75-a9fb24dde75e)

### 2. GitHub Integration

GitHub is our primary code repository. It facilitates Jenkins in the build, test, and deploy phases of the URL Shortener app. After updating and merging branches, we push the main branch back to GitHub.

To ensure seamless integration between our EC2 instance and the repository, generate and provide a token from GitHub to the EC2.



### 3. Setting Up VPC & EC2

Follow the instructions to:



### 4. EC2 Configurations

**Python Configurations**: Python facilitates various stages of the application.



**Nginx Installation**: Serves as the web server for the application.



**Jenkins Setup**: Automates the different stages of the application's lifecycle.



### 5. CloudWatch Configuration & Alarm Setting

CloudWatch tracks resource consumption, and alarms help ensure resource usage doesn't go unnoticed.

- [Install & Set Up CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/install-CloudWatch-Agent-on-EC2-Instance-fleet.html)
- [Alarm Creation in CloudWatch](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ConsoleAlarms.html)

### 6. GitHub Webhook Integration

To allow for automatic triggering of Jenkins builds upon code commits to GitHub, webhooks are utilized.



### 7. Jenkins Build Configuration

Jenkins allows for the creation of builds tailored to specific requirements.



---

## Conclusions and Recommendations

Upon evaluation of AWS's diverse instance offerings, we believe that for singular builds, our current t2.medium instance might be overkill. However, for consecutive builds, 

Issues observed:

1. Initial build didn't generate a notification, potentially due to CloudWatch's initial setup delay.

Future Optimization Areas:

- Infrastructure automation using tools like Terraform.

---

