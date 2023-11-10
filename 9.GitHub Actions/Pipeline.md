# Maven Package Workflow

This workflow will build a package using Maven and then publish it to GitHub Packages when a release is created. For more information, see [GitHub Actions - Advanced Usage](https://github.com/actions/setup-java/blob/main/docs/advanced-usage.md#apache-maven-with-a-settings-path).

```yaml
name: Maven Package

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
        SONAR_TOKEN: ${{ secrets.SONARQUBE_TOKEN }}   # Generate a token on SonarQube, add it to the secrets of this repo with the name SONAR_TOKEN (Settings > Secrets > Actions > add new repository secret)
        SONAR_HOST_URL: ${{ secrets.SONARQUBE_HOST }}   # add the URL of your instance to the secrets of this repo with the name SONAR_HOST_URL (Settings > Secrets > Actions > add new repository secret)
      with:
        args:
          -Dsonar.projectKey=Secret-Santa
          -Dsonar.java.binaries=.
