# Task 8: Grafana Installation and Dashboard Creation

## Objective

In this task, you will install Grafana on your Kubernetes (K8s) cluster using a Helm chart and create a dashboard to visualize Prometheus metrics.

## File Structure
- **```.github/workflows/```**:
  The directory is where GitHub-specific files are stored, particularly workflows for GitHub Actions.
- **```helm/grafana-values.yamll```**:  
  Configuration file of Grafana Helm chart

## Steps

1.  **Install Grafana**
    
    Install Grafana using the Bitnami Helm chart:

    ```
    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm install grafana oci://registry-1.docker.io/bitnamicharts/grafana
    ```
    Replace ```grafana``` with your desired release name.
    Refer to the Bitnami Grafana Chart Documentation for additional options.

2. **Configure Nginx Proxy on the Bastion**

    - a. Install Nginx on the Bastion

    Install Nginx using your package manager:
    ```
    sudo apt update && sudo apt install nginx -y
    ```

    - b. Configure Nginx to Proxy Grafana

    Create an Nginx configuration file for Grafana:
    ```
    sudo nano /etc/nginx/sites-available/grafana
    ```

    Add the following content:

    ``` 

    server {
        listen 80;
        server_name rs-test.cloudns.cl;

        location / {
            proxy_pass http://<grafana-service>:80;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

    server {
        listen 443 ssl;
        server_name rs-test.cloudns.cl;

        ssl_certificate /etc/letsencrypt/live/grafana.example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/grafana.example.com/privkey.pem;

        location / {
            proxy_pass http://<grafana-service>:80;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
    ```

    Enable the configuration:
    ```
    sudo ln -s /etc/nginx/sites-available/grafana /etc/nginx/sites-enabled/
    sudo systemctl reload nginx
    ```

    - c. Obtain a Let's Encrypt Certificate
    
    Install Certbot and obtain an SSL certificate:
    ```
    sudo apt install certbot python3-certbot-nginx -y
    sudo certbot --nginx -d rs-test.cloudns.cl
    ```

2. **Create a Grafana Dashboard**

    - a. Add a Data Source

        - Login to Grafana at https://rs-test.cloudns.cl/ 
        
        - Navigate to Configuration > Data Sources and add Prometheus

          - URL: http://<prometheus-service>:9090

    - b. Create a Dashboard

        - Navigate to Dashboards > New Dashboard.

        - Add panels for key metrics. For example:

            - CPU Utilization: sum(rate(container_cpu_usage_seconds_total[5m])) by (node)

            - Memory Utilization: sum(container_memory_working_set_bytes) by (node)


**Access Information**

    - Grafana URL: https://rs-test.cloudns.cl/

