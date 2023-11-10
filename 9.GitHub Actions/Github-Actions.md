# Github Actions

GitHub Actions is a powerful workflow automation and continuous integration/continuous deployment (CI/CD) tool provided by GitHub. It allows you to automate various tasks, such as building, testing, and deploying your code, directly within your GitHub repository. Workflows are defined using YAML files and can be triggered by events such as pushes, pull requests, issue comments, and more. Let's walk through a detailed example of how to set up a GitHub Actions workflow.

**Example Scenario:**

In this example, we will create a simple GitHub Actions workflow that automatically builds and tests a Python application whenever changes are pushed to the repository.

**Step 1: Create a Workflow YAML File**

Inside your GitHub repository, create a directory named `.github/workflows`. In this directory, create a YAML file (e.g., `build-test.yml`) where you'll define your workflow.

**Step 2: Define Workflow**

Add the following content to your `build-test.yml` file:

```yaml
name: Build and Test
on:
  push:
    branches:
      - main

jobs:
  build:
    name: Build and Test Python App
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout Repository
      uses: actions/checkout@v2
    
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: 3.8
        
    - name: Install Dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        
    - name: Run Tests
      run: |
        pytest
        
    - name: Upload Test Results
      uses: actions/upload-artifact@v2
      with:
        name: test-results
        path: test-reports
```

Let's break down what each part of this YAML file does:

- `name`: Name of the workflow.
- `on`: Specifies when the workflow should run. In this case, it runs when there's a push event to the `main` branch.
- `jobs`: The list of jobs that make up the workflow.
- `build`: The name of the job.
- `runs-on`: Specifies the type of runner environment. `ubuntu-latest` uses the latest version of Ubuntu.
- `steps`: The list of steps to perform in the job.
  - `actions/checkout@v2`: Checks out the repository code.
  - `actions/setup-python@v2`: Sets up the Python environment.
  - `pip install -r requirements.txt`: Installs the project dependencies.
  - `pytest`: Runs the test suite.
  - `actions/upload-artifact@v2`: Uploads the test results as an artifact.

**Step 3: Push Changes**

Commit and push the `build-test.yml` file to your repository's `main` branch.

**Step 4: Observe Workflow Execution**

After pushing the changes, visit the "Actions" tab in your GitHub repository. You'll see your workflow listed. You can click on it to view the workflow execution, including each step's logs.

Whenever you push changes to the `main` branch, GitHub Actions will automatically trigger the workflow, which will then perform the specified steps.

Remember that this is a simple example. GitHub Actions supports more advanced features such as matrix builds, environment variables, conditional steps, and deployment to various platforms. As your project and needs grow, you can customize your workflows accordingly.

# Add a VM as a Runner

To connect a virtual machine (VM) as a runner in GitHub Actions, you can use a self-hosted runner. A self-hosted runner allows you to use your own hardware, including VMs, to run GitHub Actions workflows. This is useful when you need specific software or environments that GitHub's hosted runners don't provide.

Here's a step-by-step guide on how to set up a self-hosted runner using a virtual machine:

**Step 1: Prepare the Virtual Machine**

1. Create a virtual machine (VM) with the desired operating system and necessary dependencies for your workflows.

2. Make sure the VM has internet access and can reach GitHub's services.

**Step 2: Install the GitHub Runner Software**

1. On your VM, open a terminal or command prompt.

2. Navigate to the [GitHub repository](https://github.com/actions/runner) for the self-hosted runner.

3. Download the runner package suitable for your VM's operating system. For example, if you're using Ubuntu, you can run:

   ```bash
   curl -O -L https://github.com/actions/runner/releases/download/v2.289.0/actions-runner-linux-x64-2.289.0.tar.gz
   ```

4. Extract the downloaded package:

   ```bash
   tar xzf ./actions-runner-linux-x64-2.289.0.tar.gz
   ```

**Step 3: Configure and Start the Runner**

1. Configure the runner by running the `config.sh` script:

   ```bash
   ./config.sh --url https://github.com/YOUR_USERNAME/YOUR_REPOSITORY --token YOUR_PERSONAL_ACCESS_TOKEN
   ```

   Replace `YOUR_USERNAME` with your GitHub username and `YOUR_REPOSITORY` with the repository where you want to use the self-hosted runner. You can generate a personal access token by going to your GitHub account settings.

2. Choose a name for your runner, and optionally specify labels for the runner. Labels can be useful for targeting specific runners in your workflows.

3. Start the runner:

   ```bash
   ./run.sh
   ```

The runner is now set up and connected to your GitHub repository. When you create or modify a workflow to use the self-hosted runner, GitHub Actions will automatically route jobs to this runner if they match the runner's labels.

**Step 4: Use the Self-Hosted Runner in Workflow**

In your workflow YAML file (e.g., `.github/workflows/build.yml`), you can specify the `runs-on` field to use the self-hosted runner. For example:

```yaml
jobs:
  build:
    runs-on: self-hosted  # Use the label of your self-hosted runner
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2
      # ... other steps ...
```

Remember to adjust the `self-hosted` label to match the label you configured for your self-hosted runner.

With this setup, the workflow will be executed on your self-hosted runner whenever it's triggered. You can also control the usage of self-hosted runners using labels and other workflow settings.

# Sample Pipeline for reference

An example GitHub Actions YAML file that performs Maven package, SonarQube analysis, OWASP dependency check, Docker build, and Docker image push:

```yaml


name: CI-CD Pipeline

on:
  push:
    branches:
      - master

jobs:
  build:

    runs-on: self-hosted
    permissions:
      contents: read
      packages: write

    steps:
    - uses: actions/checkout@v3
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
        server-id: github # Value of the distributionManagement/repository/id field of the pom.xml
        settings-path: ${{ github.workspace }} # location for the settings.xml file

    - name: Build with Maven
      run: mvn -B package --file pom.xml

    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    - name: Build and Push Docker Image
      run: |
        docker build -t secret-santa .
        docker tag secret-santa:latest adijaiswal/secret-santa:latest
        docker push adijaiswal/secret-santa:latest

    - name: Analyze with SonarQube
      uses: SonarSource/sonarqube-scan-action@7295e71c9583053f5bf40e9d4068a0c974603ec8
      env:
        SONAR_TOKEN: ${{ secrets.SONARQUBE_TOKEN }}   
        SONAR_HOST_URL: ${{ secrets.SONARQUBE_HOST }} 
      with:
        args:
          -Dsonar.projectKey=Secret-Santa
          -Dsonar.java.binaries=.
```

Here's what each step does:

1. **Checkout Repository**: This step checks out the repository code.

2. **Set up Java**: This step sets up the Java environment using the specified version.

3. **Build with Maven**: This step uses Maven to clean and package your project.

4. **SonarQube Analysis**: This step performs SonarQube analysis on your code. Make sure to replace `<your-sonar-project-key>` and `<your-sonar-organization>` with your actual SonarQube project key and organization.

5. **OWASP Dependency Check**: This step downloads and uses the OWASP Dependency Check tool to scan for known vulnerabilities in your dependencies.

6. **Login to Docker Hub**: This step uses your Docker Hub credentials (set as secrets) to log in to Docker Hub.

7. **Build and Push Docker Image**: This step builds a Docker image, tags it, and then pushes it to Docker Hub. Replace `your-docker-image-name` and `your-dockerhub-username` with appropriate values.

Remember to replace placeholders like `<your-sonar-project-key>`, `<your-sonar-organization>`, `your-docker-image-name`, `your-dockerhub-username`, and configure the necessary secrets in your GitHub repository settings.

Additionally, adapt the paths and commands as needed to match your project's structure and requirements.
