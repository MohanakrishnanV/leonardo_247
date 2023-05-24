**System Requirement:**

1. **Instance Type:** It is recommended to use at least a t2.medium or similar instance type, which provides a balance between cost and performance. However, depending on your specific usage and requirements, you may need a larger instance type with more resources.
2. **Operating System:** Sentry supports various operating systems, including Linux distributions like Ubuntu, CentOS, and Amazon Linux. Ensure that your EC2 instance is running a compatible Linux distribution.
3. **CPU:** A multi-core CPU is recommended to handle the processing requirements of Sentry effectively. At least 2 CPU cores should be allocated to the instance.
4. **RAM:** Sentry's memory requirements depend on the volume of events and the number of users. A minimum of 4 GB RAM is generally recommended, but it can vary depending on your usage. Consider allocating more RAM if you anticipate a large volume of events or heavy usage.
5. **Storage:** The storage requirements for Sentry depend on the number of events and attachments you expect to handle. Additionally, it's essential to allocate enough disk space for your server logs, database, and any other related files. A general recommendation is to have at least 20 GB of available disk space.
6. **Network:** Ensure that your EC2 instance has proper inbound and outbound network access. Configure security groups and network settings to allow incoming connections on the required ports for Sentry (e.g., HTTP/HTTPS).
7. **Database:** Sentry supports various database options, such as PostgreSQL and MySQL. You should have a separate database instance set up and configured to work with Sentry. It's advisable to use a managed database service like Amazon RDS or host the database on a separate EC2 instance.

Remember to consider your specific use case, anticipated traffic, and any additional services you plan to integrate with Sentry. These requirements are a general guideline, and you may need to adjust them based on your specific needs.

**Installation**

*First ensure that the security group associated with the instance allows incoming connections on ports 22 (SSH), 80 (HTTP), and 443 (HTTPS).*

**1. Install and Set Up PostgreSQL (Database):**
-   Install PostgreSQL server:
    ```shell
    sudo apt install -y postgresql
    ```
- create a new PostgreSQL database and user for Sentry:
    ```shell
    sudo -u postgres psql
    ```
- create a database and user with all Privileges
    ```sql
    CREATE DATABASE sentrydb;
    CREATE USER sentryuser WITH ENCRYPTED PASSWORD 'password';
    GRANT ALL PRIVILEGES ON DATABASE sentrydb TO sentryuser;
    \q
    ```
**2. Install and Configure Redis (Message Broker):**
- Install Redis server:
    ```shell
    sudo apt install -y redis-server
    ```
- Open the Redis configuration file for editing:
    ```shell
    sudo nano /etc/redis/redis.conf
    ```
- Locate the supervised directive and change its value to systemd:
    ```shell
    supervised systemd
    ```
- Save the file and exit the text editor. Restart Redis for the changes to take effect:
    ```shell
    sudo systemctl restart redis
    ```
**3. Create a Virtual Environment:**
- Install the venv module if it's not already installed:
    ```shell
    sudo apt install -y python3-venv
    ```
- Create a new directory for your virtual environment:
    ```shell
    mkdir ~/sentry-venv
    ```
- Create the virtual environment inside the directory:
    ```shell
    python3 -m venv ~/sentry-venv
    ```
- Activate the Virtual Environment:
    ```shell
    source ~/sentry-venv/bin/activate
    ```
**4. Install Sentry and Dependencies:**
- Installing dependencies within the virtual environment
    ```shell
    sudo apt install -y build-essential python3-dev libjpeg-dev zlib1g-dev libffi-dev libssl-dev libxml2-dev libxslt1-dev libtiff5-dev libjpeg8-dev liblcms2-dev libwebp-dev libyaml-dev libpq-dev libbz2-dev libreadline-dev libsqlite3-dev libncurses5-dev libncursesw5-dev libffi-dev libssl-dev libbz2-dev liblzma-dev zlib1g-dev uuid-dev libxml2-dev libxmlsec1-dev pkg-config
    ```
- Installing Sentry:
    ```shell
    pip install sentry
    ```
- Initialize the Sentry Configuration:
    ```shell
    sentry init
    ```
- Configure Sentry: Update the configuration file with your specific settings, such as database details, secret key, email configuration, etc
    ```shell
    nano ~/sentry-venv/sentry/sentry.conf.py
    ```
    
- Set Up Database Schema: Apply the initial migrations to set up the Sentry database schema:
    ```shell
    sentry upgrade
    ```
- Start the Sentry Services: Start the Sentry web service and worker processes.
    ```shell
    sentry run web
    sentry run worker
    ```

Remember to activate the virtual environment `(source ~/sentry-venv/bin/activate)` every time you want to run Sentry or its related commands.

- Creating `sentry run web` and `sentry run worker` as **systemd services**
    ```shell
    sudo nano /etc/systemd/system/sentry-web.service
    ```
    ```shell
    [Unit]
    Description=Sentry Service
    After=network.target

    [Service]
    User=ubuntu
    Group=ubuntu
    WorkingDirectory=/home/ubuntu/sentry-venv/bin
    ExecStart=/bin/bash -c "source /home/ubuntu/sentry-venv/bin/activate && sentry run web"
    Environment="PATH=/home/ubuntu/sentry-venv/bin"
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```

    ```shell
    sudo nano /etc/systemd/system/sentry-worker.service
    ```
    
    ```shell
    [Unit]
    Description=Sentry Service
    After=network.target

    [Service]
    User=ubuntu
    Group=ubuntu
    WorkingDirectory=/home/ubuntu/sentry-venv/bin
    ExecStart=/bin/bash -c "source /home/ubuntu/sentry-venv/bin/activate && sentry run worker"
    Environment="PATH=/home/ubuntu/sentry-venv/bin"
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```
    
    ```shell
    sudo nano /etc/systemd/system/sentry-cron.service
    ```
    
    ```shell
    [Unit]
    Description=Sentry Service
    After=network.target

    [Service]
    User=ubuntu
    Group=ubuntu
    WorkingDirectory=/home/ubuntu/sentry-venv/bin
    ExecStart=/bin/bash -c "source /home/ubuntu/sentry-venv/bin/activate && sentry run cron"
    Environment="PATH=/home/ubuntu/sentry-venv/bin"
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```
    
- Start the services and check for status
    ```shell
    sudo systemctl start sentry-web
    ```
    ```shell
    sudo systemctl start sentry-worker
    ```
    ```shell
    sudo systemctl start sentry-cron
    ```
    ```shell
    sudo systemctl status sentry-web
    ```
    ```shell
    sudo systemctl status sentry-worker
    ``` 
    ```shell
    sudo systemctl status sentry-cron
    ``` 
    ```shell
    sudo systemctl enable sentry-web
    ```
    ```shell
    sudo systemctl enable sentry-worker
    ``` 
    ```shell
    sudo systemctl enable sentry-cron
    ``` 
