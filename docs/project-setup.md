# Project Setup

This document provides step-by-step instructions for setting up the local and AWS environments for our AI-Human Alignment Researcher project.

## Local Environment Setup

1. **Project Structure**: Organize your project with this structure:

    ```
    project_root/
    ├── research/
    ├── infrastructure/
    ├── data/
    └── tests/
    ```

2. **Python Environment**:

    - Install `virtualenv` using pip: `pip install virtualenv`
    - Navigate to your project directory: `cd project_root`
    - Create a new virtual environment: `virtualenv venv`
    - Activate the environment: `source venv/bin/activate` (on Unix or MacOS) or `.\venv\Scripts\activate` (on Windows)
    - Install necessary libraries: `pip install tensorflow boto3 pandas jupyter`

3. **Docker**:
> Multi-stage builds are a great way to keep the size of your final image down. They can also reduce build times if you properly leverage caching for dependencies that don't change often.

    - Install Docker from [https://www.docker.com/get-started](https://www.docker.com/get-started)
    - Create a multi-stage `Dockerfile` in your project root:

        ```dockerfile
        # Use an official Python runtime as a parent image
        FROM python:3.9 as base

        # Set the working directory
        WORKDIR /app

        # Copy the current directory contents into the container at /app
        COPY . /app

        # Install any needed packages specified in requirements.txt
        RUN pip install --no-cache-dir -r requirements.txt

        # Use a slim version for the final image
        FROM python:3.9-slim as final

        WORKDIR /app

        # Copy only the necessary files from the base image
        COPY --from=base /app .

        # Make port 8888 available to the world outside this container
        EXPOSE 8888

        # Run jupyter notebook when the container launches
        CMD ["jupyter", "notebook", "--ip=0.0.0.0", "--allow-root"]
        ```

    - Build your Docker image: `docker build -t my-research-image .`
    - Run your Docker container: `docker run -p 8888:8888 my-research-image`

## Version Control and CI/CD

1. **Git**:

    - Install Git from [https://git-scm.com/downloads](https://git-scm.com/downloads)
    - Initialize a Git repository in your project root: `git init`
    - Add all files to the repository: `git add .`
    - Commit the changes: `git commit -m "Initial commit"`

2. **GitHub Actions**:

    - Create a new file in your project root: `.github/workflows/main.yml`
    - Add this content:

        ```yaml
        name: CI

        on:
          push:
            branches: [ main ]

        jobs:
          build:
            runs-on: ubuntu-latest
            steps:
            - uses: actions/checkout@v2
            - name: Set up Python 3.9
              uses: actions/setup-python@v2
              with:
                python-version: 3.9
            - name: Install dependencies
              run: |
                python -m pip install --upgrade pip
                pip install -r requirements.txt
            - name: Run tests
              run: |
                pytest
        ```

    - Push your changes to GitHub: `git push origin main`

## Data Management Locally and on AWS Free Tier

1. **Local Data Processing**:

    - Use Pandas for data processing: `import pandas as pd`

2. **AWS Free Tier**:

    - Create an AWS account and access the [AWS Management Console](https://aws.amazon.com/console/)
    - For data ingestion and ETL jobs, use [AWS Glue](https://aws.amazon.com/glue/)
    - For data storage, use [AWS S3](https://aws.amazon.com/s3/)
    - For database services, use [AWS RDS](https://aws.amazon.com/rds/) for structured data or [AWS DynamoDB](https://aws.amazon.com/dynamodb/) for unstructured data

    Remember, usage of AWS Free Tier services is limited. Monitor your usage to avoid exceeding these limits.

## Security and Monitoring Locally and on AWS

1. **Local Security**:

    - Install Vault by HashiCorp: `docker pull vault`
    - Run Vault in development mode: `docker run --cap-add=IPC_LOCK -e 'VAULT_DEV_ROOT_TOKEN_ID=myroot' -p 8200:8200 vault`

2. **AWS Free Tier**:

    - For security, use [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
    - For monitoring and logging, use [AWS CloudWatch](https://aws.amazon.com/cloudwatch/)

## Jupyter Notebook/Lab

- Install Jupyter Notebook/Lab in your local Python environment: `pip install jupyterlab`
- Run Jupyter: `jupyter lab`

## OpenAI API for ChatGPT

- Get your API key from OpenAI's website
- Install the OpenAI Python client: `pip install openai`
- Use the API in your code:

    ```python
    import openai

    openai.api_key = "your-api-key"

    response = openai.Completion.create(
      engine="text-davinci-003",
      prompt="Translate the following English text to French: '{}'",
      max_tokens=60
    )

    print(response.choices[0].text.strip())
    ```
