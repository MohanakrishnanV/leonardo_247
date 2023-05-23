Custom GIT exporter in Python to export branch name and commit ID of a `/src` folder through Prometheus:

1. Install the necessary packages:
   ```shell
   pip install prometheus-client GitPython
   ```

2. Create a Python script that uses the `prometheus_client` library to expose the Git branch and commit ID as Prometheus metrics. Here's an example script:

   ```python
   #!/usr/bin/python3

   import git
   import time
   from prometheus_client import CollectorRegistry, Gauge, start_http_server

   #Function to retrieve the commit ID and branch name
   def get_commit_info(repo_path):
       repo = git.Repo(repo_path)
       commit_id = repo.head.object.hexsha
       branch_name = repo.active_branch.name
       return commit_id, branch_name

   #Initialize Prometheus metric registry
   registry = CollectorRegistry()

   #Define custom metrics
   commit_metric = Gauge('commit_id', 'Current commit ID', ['commit_id'], registry=registry)
   branch_metric = Gauge('branch_name', 'Current branch name', ['branch_name'], registry=registry)

   #Retrieve commit ID and branch name
   repo_path = '/src'
   commit_id, branch_name = get_commit_info(repo_path)

   #Start Prometheus HTTP server to expose metrics
   start_http_server(8000, registry=registry)

   #Continuously update metrics every 5 seconds
   while True:
       # Clear previous metric values
       commit_metric.clear()
       branch_metric.clear()

       # Retrieve updated commit ID and branch name
       commit_id, branch_name = get_commit_info(repo_path)

       # Set metric values
       commit_metric.labels(commit_id=commit_id).set(1)
       branch_metric.labels(branch_name=branch_name).set(1)

       # Delay for 5 seconds
       time.sleep(5)
   ```

   This script creates two Prometheus metrics (`git_branch` and `git_commit`) and updates their values with the current Git branch and commit ID of the `/src`. It also starts a Prometheus HTTP server on port 8000 to expose the metrics.

3. Run the Python script on the EC2 instance using a cron job or a systemd service. For example, you can create a systemd service file `/etc/systemd/system/git-exporter.service` with the following content:

   ```shell
   [Unit]
   Description=Git exporter
   After=network.target

   [Service]
   User=mohan
   Group=mohan
   WorkingDirectory=/home/mohan/docker/
   ExecStart=/usr/bin/python3 /home/mohan/docker/git_exporter.py
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

   This service file runs the Python script `/home/mohan/docker/git_exporter.py` as the `mohan` user and ensures that the script is always restarted if it crashes. You can enable and start the service using the following commands:

   ```shell
   systemctl enable git-exporter
   systemctl start git-exporter
   ```

4. Configure Prometheus to scrape the metrics from the EC2 instance. Add the following job to the Prometheus configuration file (`/etc/prometheus/prometheus.yml`):

   ```yaml
   - job_name: git_exporter
     scrape_interval: 1m
     static_configs:
       - targets:
           - ec2-instance-ip:8000
     metrics_path: /
   ```

   This job configures Prometheus to scrape metrics from the HTTP server running on the EC2 instance IP address on port 8000. The `scrape_interval` specifies how often Prometheus should scrape the metrics. The `metrics_path` specifies the path where the metrics are exposed (in this case, the root path `/`).

5. Restart Prometheus to reload the configuration file and start scraping the metrics:

   ```shell
   systemctl restart prometheus
   ```

With these steps, you should be able to export the Git branch and commit ID of the `/src` folder from the EC2 instance through Prometheus. You can visualize the metrics using Prometheus' built-in graphing features or use a third-party visualization tool like Grafana.
