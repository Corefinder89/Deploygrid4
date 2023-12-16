# Deploy grid 4 on cloud premises using Pulumi

## Installation

Make sure you have Pulumi installed on your machine. You can install it using pip:

```commandline
pip install pulumi
```
## Create a new Pulumi project

Create a new directory for your Pulumi project and navigate into it. Then, run the following command to initialize a 
new Pulumi project:

```commandline
pulumi new kubernetes-python
```
Follow the prompts to configure your project.

## Write Pulumi code

Replace the contents of `__main__.py` in your project directory with the following Pulumi code:

```python
import pulumi
import pulumi_kubernetes as k8s
from pulumi_digitalocean import kubernetes

# Digital Ocean Configuration
config = pulumi.Config()
token = config.require_secret("do_token")

# Create a Digital Ocean Kubernetes Cluster
cluster = kubernetes.Cluster(
    "my-cluster",
    provider_token=token,
    node_pool={
        "name": "worker-pool",
        "size": "s-2vcpu-2gb",
        "nodeCount": 3,
    },
)

# Deploy Selenium Hub
selenium_hub = k8s.core.v1.Pod(
    "selenium-hub",
    metadata={
        "name": "selenium-hub",
        "labels": {"app": "selenium-hub"},
    },
    spec={
        "containers": [
            {
                "name": "selenium-hub",
                "image": "selenium/hub:4.1.2",
                "ports": [{"containerPort": 4442}],
            }
        ]
    },
)

# Deploy Selenium Chrome Node
selenium_chrome_node = k8s.core.v1.Pod(
    "selenium-chrome-node",
    metadata={
        "name": "selenium-chrome-node",
        "labels": {"app": "selenium-chrome-node"},
    },
    spec={
        "containers": [
            {
                "name": "selenium-chrome-node",
                "image": "selenium/node-chrome:4.1.2",
                "ports": [{"containerPort": 5555}],
                "env": [
                    {"name": "HUB_HOST", "value": "selenium-hub"},
                    {"name": "HUB_PORT", "value": "4442"},
                ],
            }
        ]
    },
)

# Deploy Selenium Firefox Node
selenium_firefox_node = k8s.core.v1.Pod(
    "selenium-firefox-node",
    metadata={
        "name": "selenium-firefox-node",
        "labels": {"app": "selenium-firefox-node"},
    },
    spec={
        "containers": [
            {
                "name": "selenium-firefox-node",
                "image": "selenium/node-firefox:4.1.2",
                "ports": [{"containerPort": 5555}],
                "env": [
                    {"name": "HUB_HOST", "value": "selenium-hub"},
                    {"name": "HUB_PORT", "value": "4442"},
                ],
            }
        ]
    },
)

# Export the kubeconfig for external access
pulumi.export("kubeconfig", cluster.kube_config_raw)
```

## Configure digital ocean token

Set your Digital Ocean API token as a secret in the Pulumi configuration:

```commandline
pulumi config set do_token <your-digital-ocean-token> --secret
```

## Deploy the infrastructure

Run the following command to deploy your Pulumi stack:

```commandline
pulumi up
```
Pulumi will ask for confirmation before making changes. Type `yes` to proceed.

## Access selenium grid

After the deployment is complete, you can access the Selenium Grid console through the Selenium Hub service. Retrieve the external IP of the Selenium Hub service from the Digital Ocean dashboard, and open a browser to `http://<external-ip>:4442/grid/console`.
