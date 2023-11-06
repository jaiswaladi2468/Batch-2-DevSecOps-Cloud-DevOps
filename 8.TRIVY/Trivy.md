# TRIVY



## 1. Introduction to Trivy

Trivy is a vulnerability scanner designed for containers and other artifacts such as filesystems. It uses vulnerability databases like CVEs (Common Vulnerabilities and Exposures) to identify potential security issues within your container images or filesystems. Trivy supports various image formats including Docker, OCI (Open Container Initiative), and more.

## 2. Installing Trivy

Trivy can be installed on various platforms, including Linux, macOS, and Windows. The installation process might vary depending on your operating system. Below are the general steps:

1. **Linux** (example using curl):
   ```bash
   curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
   ```

2. **macOS** (example using Homebrew):
   ```bash
   brew install aquasecurity/trivy/trivy
   ```

3. **Windows** (example using Scoop):
   ```bash
   scoop install trivy
   ```

## 3. Scanning Docker Images

Trivy can scan Docker images directly from various sources, such as local Docker daemons, remote registries, and more. Here's a basic example of scanning a Docker image:

```bash
trivy image <image-name>
```

Replace `<image-name>` with the name of the Docker image you want to scan.

## 4. Scanning Filesystems

Trivy can also scan local filesystems, which is useful for scanning files and directories. Here's an example:

```bash
trivy fs <path-to-directory>
```

Replace `<path-to-directory>` with the actual path of the directory you want to scan.

## 5. Scan Output Formats

Trivy provides different output formats for its scan results, including "table", "json", "template", "yaml", and "junit". The default format is "table". You can specify the output format using the `--format` flag.

## 6. Examples

### Scanning a Docker Image

```bash
trivy image alpine:latest
```

This command scans the "alpine:latest" Docker image for vulnerabilities.

### Scanning a Local Directory

```bash
trivy fs /path/to/directory
```

This command scans the specified local directory for vulnerabilities.

### Generating JSON Output

```bash
trivy image --format json ubuntu:latest > scan_results.json
```

This command scans the "ubuntu:latest" Docker image and saves the results in JSON format to the "scan_results.json" file.

# Important Links & Commands 

---

## Installation

To install Trivy, follow the instructions provided in the official installation documentation: [Trivy Installation Guide](https://aquasecurity.github.io/trivy/v0.18.3/installation/).

## Sample Project to Scan

You can use the following example project to demonstrate Trivy's capabilities: [Shopping-Cart Sample Project](https://github.com/jaiswaladi246/Shopping-Cart.git).

## Scanning Filesystems

To scan a local directory for vulnerabilities in the filesystem, you can use the following Trivy commands:

```bash
# Scan a directory with severity filter
trivy fs --severity CRITICAL,HIGH /home/ubuntu/Shopping-Cart/

# Scan a directory without severity filter
trivy fs /home/ubuntu/Shopping-Cart/

# Scan a directory and save results in JSON format
trivy fs --format json -o report.json /home/ubuntu/Shopping-Cart/
```

## Scanning Docker Images

Trivy can also scan Docker images for vulnerabilities. Here are examples of scanning Docker images using Trivy:

```bash
# Scan a Docker image
trivy image adijaiswal/shopping-cart:latest

# Scan a Docker image and save results in JSON format
trivy image -f json -o docker-report.json adijaiswal/shopping-cart

# Scan a Docker image with severity filter
trivy image --severity HIGH adijaiswal/shopping-cart
```

Remember to replace `adijaiswal/shopping-cart:latest` with the actual Docker image name you want to scan.

---
## TRIVY Docker Setup

---TRIVY INSTALLATION STEPS---

sudo apt-get install wget apt-transport-https gnupg lsb-release

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list

sudo apt-get update

sudo apt-get install trivy -y




