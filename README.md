# E-commerce Product Assistant

This project is an AI-powered e-commerce product assistant that helps users find products, get product information, and receive recommendations. It uses a combination of web scraping, retrieval-augmented generation (RAG), and a large language model (LLM) to provide intelligent and conversational product assistance.

## Project Structure

```
├── .dockerignore
├── .env.copy
├── .gitignore
├── .python-version
├── Dockerfile
├── get_lib_versions.py
├── pyproject.toml
├── README.md
├── requirements.txt
├── scrapper_ui.py
├── .git/
├── .github/
│   └── workflows/
│       ├── deploy.yml
│       └── infra.yml
├── data/
│   └── product_reviews.csv
├── infra/
│   └── eks-with-ecr.yaml
├── k8/
│   ├── deployment.yaml
│   └── service.yaml
├── notebook/
│   ├── NIPS-2017-attention-is-all-you-need-Paper.pdf
│   └── test.ipynb
├── prod_assistant/
│   ├── config/
│   │   └── config.yaml
│   ├── etl/
│   │   ├── data_ingestion.py
│   │   └── data_scrapper.py
│   ├── evaluation/
│   │   └── ragas_eval.py
│   ├── exception/
│   │   └── custom_exception.py
│   ├── logger/
│   │   └── custom_logger.py
│   ├── mcp_servers/
│   │   ├── client.py
│   │   └── product_search_server.py
│   ├── prompt_library/
│   │   └── prompts.py
│   ├── retriever/
│   │   └── retrieval.py
│   ├── router/
│   │   └── main.py
│   ├── utils/
│   │   ├── config_loader.py
│   │   └── model_loader.py
│   └── workflow/
│       ├── agentic_rag_workflow.py
│       ├── agentic_workflow_with_mcp_websearch.py
│       ├── agentic_workflow_with_mcp.py
│       └── normal_generation_workflow.py
├── static/
│   └── style.css
├── templates/
│   └── chat.html
└── test/
```

### Key Files and Folders

| File/Folder | Description |
|---|---|
| `scrapper_ui.py` | A Streamlit application for scraping product data from e-commerce websites. |
| `prod_assistant/` | The core application code. |
| `prod_assistant/etl/` | Handles data extraction, transformation, and loading (ETL). |
| `prod_assistant/etl/data_scrapper.py` | Scrapes product data from Flipkart. |
| `prod_assistant/etl/data_ingestion.py` | Ingests the scraped data into an AstraDB vector store. |
| `prod_assistant/workflow/` | Contains the agentic RAG workflows. |
| `prod_assistant/workflow/agentic_rag_workflow.py` | Implements the core agentic RAG workflow using LangGraph. |
| `prod_assistant/router/main.py` | The FastAPI application that serves the chat interface and API. |
| `Dockerfile` | For containerizing the application. |
| `requirements.txt` | Python dependencies. |
| `pyproject.toml` | Project metadata and build system configuration. |
| `.github/workflows/` | GitHub Actions workflows for CI/CD. |
| `infra/` | Infrastructure as Code (IaC) for deployment. |
| `k8/` | Kubernetes manifests for deployment. |

## Getting Started

### Prerequisites

- Python 3.10 or higher
- An AstraDB account
- A Google AI API key

### Installation

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/your-username/e_commerce_bot.git
    cd e_commerce_bot
    ```

2.  **Create and activate a virtual environment:**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
    ```

3.  **Install the dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Set up your environment variables:**
    - Copy the `.env.copy` file to a new file named `.env`.
    - Open the `.env` file and add your AstraDB and Google AI credentials:
      ```
      GOOGLE_API_KEY="your-google-api-key"
      ASTRA_DB_API_ENDPOINT="your-astradb-api-endpoint"
      ASTRA_DB_APPLICATION_TOKEN="your-astradb-application-token"
      ASTRA_DB_KEYSPACE="your-astradb-keyspace"
      ```

## Usage

1.  **Scrape product data:**
    - Run the Streamlit scraper UI:
      ```bash
      streamlit run scrapper_ui.py
      ```
    - Open your browser to the provided URL (usually `http://localhost:8501`).
    - Enter the product names you want to scrape and click "Start Scraping".
    - Once the scraping is complete, click "Store in Vector DB (AstraDB)" to ingest the data.

2.  **Start the chat application:**
    - Run the FastAPI application:
      ```bash
      uvicorn prod_assistant.router.main:app --reload
      ```
    - Open your browser to `http://localhost:8000`.
    - You can now chat with the e-commerce product assistant.

## Deployment

This project is designed to be deployed to a Kubernetes cluster on AWS (Amazon EKS). The deployment process is automated using GitHub Actions.

### Deployment Architecture

The deployment architecture consists of the following components:

-   **Amazon ECR (Elastic Container Registry):** Stores the Docker images of the application.
-   **Amazon EKS (Elastic Kubernetes Service):** A managed Kubernetes service to run the containerized application.
-   **GitHub Actions:** For CI/CD (Continuous Integration/Continuous Deployment).

### Deployment Files

Here's a breakdown of the files used for deployment:

#### `Dockerfile`

This file defines the Docker image for the application. It does the following:

-   Starts from a Python 3.11 base image.
-   Installs Git.
-   Copies the application code and dependencies.
-   Installs the Python dependencies using `pip`.
-   Exposes port 8000.
-   Starts the application using `uvicorn` and the `product_search_server.py`.

#### `infra/eks-with-ecr.yaml`

This is a CloudFormation template that provisions the necessary AWS infrastructure:

-   **VPC:** A new Virtual Private Cloud (VPC) with public subnets.
-   **EKS Cluster:** An Amazon EKS cluster named `product-assistant-cluster-latest`.
-   **EKS Node Group:** A managed node group of `t3.medium` instances for the EKS cluster.
-   **ECR Repository:** An Amazon ECR repository named `product-assistant` to store the Docker images.
-   **IAM Roles:** Necessary IAM roles for the EKS cluster and node group.

#### `.github/workflows/infra.yml`

This GitHub Actions workflow automates the infrastructure provisioning:

-   **Trigger:** Manually triggered (`workflow_dispatch`).
-   **Jobs:**
    -   `provision`: Checks out the code, configures AWS credentials, and deploys the CloudFormation stack defined in `infra/eks-with-ecr.yaml`.

#### `k8/deployment.yaml`

This Kubernetes manifest defines the Deployment for the `product-assistant` application:

-   **Replicas:** Specifies 2 replicas of the application pod.
-   **Container Image:** Uses the `product-assistant:latest` image from the ECR repository.
-   **Ports:** Exposes container port 8000.
-   **Environment Variables:** Loads sensitive information (API keys and database credentials) from a Kubernetes Secret named `product-assistant-secrets`.

#### `k8/service.yaml`

This Kubernetes manifest defines a Service to expose the `product-assistant` application:

-   **Type:** `LoadBalancer` - This will create an external load balancer in AWS to expose the service to the internet.
-   **Selector:** Selects the pods with the label `app: product-assistant`.
-   **Ports:** Maps port 80 of the load balancer to port 8000 of the application pods.

#### `.github/workflows/deploy.yml`

This GitHub Actions workflow automates the application deployment:

-   **Trigger:** On every push to the `main` branch.
-   **Jobs:**
    -   `deploy`: 
        1.  Checks out the code.
        2.  Configures AWS credentials.
        3.  Logs in to Amazon ECR.
        4.  Builds the Docker image, tags it with a unique tag and `latest`, and pushes it to the ECR repository.
        5.  Sets up `kubectl`.
        6.  Updates the `kubeconfig` to connect to the EKS cluster.
        7.  Creates or updates a Kubernetes Secret named `product-assistant-secrets` with the necessary API keys.
        8.  Applies the Kubernetes manifests from the `k8/` directory.
        9.  Patches the deployment with the new image tag to trigger a rolling update.
        10. Verifies the deployment rollout.

### Deployment Steps

1.  **Provision the infrastructure:**
    -   Manually trigger the "Provision Infra (EKS + ECR)" workflow in GitHub Actions.
    -   This will create the EKS cluster, ECR repository, and other necessary resources.

2.  **Deploy the application:**
    -   Push a change to the `main` branch.
    -   This will trigger the "Deploy App to EKS" workflow, which will build and deploy the latest version of the application to the EKS cluster.
