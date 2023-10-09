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

Despite our preparations, the QA test was not entirely successful. We found that 8 of the 14,000 requests failed. Additionally, our CPU utilization skyrocketed to 99.7%.

![image](https://github.com/atlas-lion91/Blitz_2/assets/140761974/aaf96a7d-6082-43b4-8bb5-ddc62e8cd38c)


### Reliability Testing and Insights

Running a heavy load of 14,000 requests while concurrently executing the `stress-ng` script stretched our server's capabilities. This test was designed to gauge how well our server could cope under extreme conditions.

#### Testing on t2.medium

Examples of CPU utilization on a t2.medium instance (2 CPUs) when subjected to stress:

- ![image](https://github.com/atlas-lion91/Blitz_2/assets/140761974/7322fe05-f123-412a-8ce9-2fa5483d2351)

Our observation was that the two CPUs were nearly saturated at 99% even without any active requests or Jenkins builds. We believe we might need to upgrade our resources, given the three-tier structure of our application.

#### Testing on t2.xlarge

During the stress test, where the command `sudo nice -n -20 stress-ng --cpu 2` was executed, and multiple Jenkins builds were simultaneously running on a t2.xlarge instance, the following observations were made regarding CPU utilization:

- Three out of the four CPUs were operating at approximately 75% capacity or higher.
- One CPU was running at around 13% capacity.

Considering the current resource usage, it appears that even with four CPUs, the system may struggle to handle the additional stress of accommodating a minimum of 14,000 requests.



Considering our server's multi-tier architecture, encompassing the Web, Application, and Data tiers, simply doubling the CPUs might not suffice.

---


### 1. Infrastructure Blueprint 

   ![Blitz 2 drawio](https://github.com/atlas-lion91/Blitz_2/assets/140761974/35a61ce2-c246-4f27-8f75-a9fb24dde75e)


---

## Conclusions and Recommendations

### Enhancing CPU Resources for Improved Performance

To enhance system performance, we have two recommended options:

### Increase CPU Cores:

Increase the number of CPU cores to 8 by choosing between the `t2.2xlarge` or `c5.2xlarge` instance types. The key difference lies in their memory configurations: `t2.2xlarge` offers 32 GiB, while `c5.2xlarge` provides 16 GiB of memory. Given that this instance hosts our data tier, we strongly recommend the `t2.2xlarge` instance with 32 GiB of memory for optimal performance.

### Separate Application and Data Tiers:

Consider segregating the application tier and data tier into separate instances. In this case:

- Deploy the web tier on an instance equipped with 4 CPUs.
- Place the application and data tiers on a separate instance, which can suffice with 2 CPUs.

This approach not only enhances security by isolating the application and web tiers but also optimizes resource allocation. However, please be aware that adding an additional instance will result in incremental costs.

Weigh the advantages of both options against your specific requirements and budget constraints to determine the most suitable approach for your infrastructure.

### Infrastructure Automation:
Consider implementing infrastructure automation using tools like Terraform. This will help streamline provisioning, scaling, and management of your infrastructure.


---

