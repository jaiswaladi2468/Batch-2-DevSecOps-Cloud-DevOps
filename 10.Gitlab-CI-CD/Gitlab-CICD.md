# GITLAB CI/CD

GitLab provides a robust platform for automating the build, testing, and deployment process of your software projects. Here's a general overview of how you might set up a full CI/CD pipeline in GitLab:

1. **Repository Setup:**
   Create a GitLab repository for your project if you haven't already. Make sure your project contains a `.gitlab-ci.yml` file at the root. This file will define your pipeline configuration using YAML syntax.

2. **CI/CD Pipeline Configuration (`.gitlab-ci.yml`):**
   The `.gitlab-ci.yml` file is the heart of your CI/CD pipeline configuration. It defines the stages, jobs, and scripts to be executed at each stage. Here's a basic template:

   ```yaml
   stages:
     - build
     - test
     - deploy

   build_job:
     stage: build
     script:
       - # Your build commands here

   test_job:
     stage: test
     script:
       - # Your test commands here

   deploy_job:
     stage: deploy
     script:
       - # Your deployment commands here
     only:
       - master  # Deploy only when changes are pushed to the master branch
   ```

3. **Runners:**
   GitLab Runners are agents responsible for running CI/CD jobs defined in your `.gitlab-ci.yml` file. You can set up runners on your own infrastructure or use GitLab's shared runners. Runners execute the commands specified in your pipeline stages on the environment of your choice.

4. **Variables and Secrets:**
   Store sensitive information like API keys, access tokens, or passwords as protected variables or in GitLab's CI/CD secret vault. Refer to these secrets in your pipeline configuration rather than hardcoding them.

5. **Testing:**
   Within the `test` stage of your pipeline, you can run various tests (unit tests, integration tests, etc.) to ensure the quality of your code.

6. **Deployment:**
   The `deploy` stage is where you can automate the deployment of your application to your target environment, whether it's a staging or production environment. Common deployment strategies include blue-green deployments, canary releases, or simple automatic deployments.

7. **Monitoring and Notifications:**
   Configure notifications to alert your team about the status of your CI/CD pipeline. GitLab supports email notifications, integrations with chat tools like Slack, and more.

8. **Versioning and Tagging:**
   Consider using versioning and tagging in your repository to keep track of different releases. This can be useful when deploying specific versions of your application.


# Adding a self hosted Runner

Adding a self-hosted runner to GitLab allows you to execute CI/CD jobs on your own infrastructure, providing more control over the environment and resources. Here's how you can set up a self-hosted runner in GitLab:

1. **Prerequisites:**
   - A machine (physical or virtual) to serve as the runner.
   - GitLab account with appropriate permissions to register runners.

2. **Install GitLab Runner:**
   You need to install the GitLab Runner software on the machine you want to use as a self-hosted runner. The installation steps might vary based on your operating system. Refer to the official documentation for installation instructions: https://docs.gitlab.com/runner/install/

3. **Register the Runner:**
   After installing the GitLab Runner, you need to register it with your GitLab project. Run the registration command on your runner machine:

   ```bash
   sudo gitlab-runner register
   ```

   Follow the prompts to configure the runner:


---

### Step 1
Copy and paste the following command into your command line to register the runner.

```
gitlab-runner register  --url https://gitlab.com  --token asasas-Bu6YESU7oL2HjcLxz2
```

The runner token `asasas-Bu6YESU7oL2HjcLxz2` displays only for a short time and is stored in the `config.toml` after you register the runner. It will not be visible once the runner is registered.

### Step 2
Choose an executor when prompted by the command line. Executors run builds in different environments. Not sure which one to select?

### Step 3 (optional)
Manually verify that the runner is available to pick up jobs.

```
gitlab-runner run
```

---

4. **Configure Runner Tags (Optional):**
   You can assign tags to your self-hosted runner during registration. Tags help you target specific runners for certain jobs in your `.gitlab-ci.yml` file.

5. **Start the Runner:**
   After registration, start the runner service:

   ```bash
   sudo gitlab-runner start
   ```

6. **Update `.gitlab-ci.yml`:**
   In your project's `.gitlab-ci.yml` file, you can now use the tags you assigned to the self-hosted runner to specify where each job should run. For example:

   ```yaml
   test_job:
     tags:
       - my-self-hosted-runner
     script:
       - # Your test commands here
   ```

7. **Verify and Monitor:**
   Go to your GitLab project's CI/CD settings to verify that the runner is registered and online. You can also monitor the runner's status and job execution logs from the GitLab UI.

8. **Keep the Runner Updated:**
   Regularly update the GitLab Runner to ensure you have the latest features, bug fixes, and security patches.

Remember that self-hosted runners require maintenance and monitoring to ensure their availability and security. Always keep the runner software updated and ensure that the machine hosting the runner is properly maintained.

## Sample Pipeline


```yaml
stages:
  - compile
  - test
  - build
  - deploy

compile-job:
  tags:
    - linux
  stage: compile
  script:
    - echo "Compiling the code..."
    - mvn compile

unit-test-job:
  tags:
    - linux
  stage: test
  script:
    - echo "Running unit tests... This will take about 60 seconds."
    - mvn test

build-job:
  tags:
    - linux
  stage: build
  script:
    - echo "Compiling the code..."
    - mvn install
  artifacts:
    paths:
      - target/*

deploy-job:
  tags:
    - linux
  stage: deploy
  environment: production
  script:
    - echo "Deploying application..."
    - echo "Application successfully deployed."
```

