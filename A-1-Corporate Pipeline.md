# Corporate Pipeline

```groovy

pipeline {
    agent { label 'VM' }
    tools {
        maven 'Maven3'
    }
    //Comment added
    environment {
        sonarEnv = 'SonarQube'
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }

    stages {
        stage('Compile') {
            steps {
                withMaven(globalMavenSettingsConfig: 'MavenSettings' ,jdk: 'OpenJDK 1.8.181',  maven: 'Maven3', options: [jacocoPublisher(disabled: true), junitPublisher(disabled: true, healthScaleFactor: 1.0)]) {
                    // Compile the source code
                    sh 'mvn -B clean compile -Dmaven.test.skip=true'
                    // Compile the tests
                    sh 'mvn -B clean compile'
                }
            }
        }

        stage('Unit Tests') {
            steps {
                withMaven(globalMavenSettingsConfig: 'MavenSettings' ,jdk: 'OpenJDK 1.8.181',  maven: 'Maven3') {
                    // Execute Unit-Tests
                    sh 'mvn -B test org.jacoco:jacoco-maven-plugin:prepare-agent install'
                }
            }
        }

        stage('Package') {
            steps {
                withMaven(globalMavenSettingsConfig: 'MavenSettings' ,jdk: 'OpenJDK 1.8.181',  maven: 'Maven3', options: [jacocoPublisher(disabled: true), junitPublisher(disabled: true, healthScaleFactor: 1.0)]) {
                    sh 'mvn -B install -Dmaven.test.skip.exec'
                }
            }
        }

        stage('OSS Compliance') {
            steps {
                withMaven(globalMavenSettingsConfig: 'MavenSettings', jdk: 'OpenJDK 1.8.181', maven: 'Maven3', options: [jacocoPublisher(disabled: true), junitPublisher(disabled: true, healthScaleFactor: 1.0)]) {
                    dependencyCheck additionalArguments: '', odcInstallation: 'OWASP-dependency'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }

        stage('Code Quality Check') {
            steps {
                withMaven(globalMavenSettingsConfig: 'MavenSettings', jdk: 'OpenJDK 1.8.181', maven: 'Maven3', options: [jacocoPublisher(disabled: true), junitPublisher(disabled: true, healthScaleFactor: 1.0)]) {
                    withSonarQubeEnv(sonarEnv) {
                        sh 'mvn -B -Dsonar.host.url=xxxxxxxxxxxx -Dsonar.login=xxxxxxxxxxxxx -Dsonar.exclusions=src/main/java/com/xxx/to/**,src/main/java/com/appcore/to/**,src/main/java/com/appcore/dataaccess/model/**,src/main/java/com/SpringBootApp.java,src/main/java/com/general/** -DSkipTests=true sonar:sonar -Dsonar.links.ci=${BUILD_URL} -Dsonar.dependencyCheck.reportPath=target/dependency-check-report.xml -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html -Dsonar.jacoco.reportPath=target/jacoco.exec'
                    }
                }
            }
        }

        stage('Build and upload image') {
            // Switch to external VM where docker is installed
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    docker.withTool('docker') {
                        docker.withRegistry('http://docker-registry-xxxx.com', 'xxxxxxxxxxxxxxxxx-b5a0-5e4b2e63c889') {
                            def customImage = docker.build("${pom.artifactId}:${pom.version}", './server')
                            customImage.push('latest')
                            customImage.push("store-${pom.version}")
                        }
                    }
                }
            }
        }

        stage('Update Image Stream DEV') {
            steps {
                // Update the image stream of the given image
                script {
                    withEnv(["PATH+OC=${tool 'OpenShiftv3.11.0'}"]) {
                        withCredentials([string(credentialsId: 'demo-dev', variable: 'TOKEN')]) {
                            sh 'oc login https://console-openshift-xxxxxxxxxxxxxxx.com:6443 --token=$TOKEN --insecure-skip-tls-verify'
                        }
                        sh 'oc get projects'
                        sh 'oc project demo-dev'
                        sh 'oc import-image store'
                    }
                }
            }
        }

        stage('Deployment to DEV') {
            steps {
                script {
                    withEnv(["PATH+OC=${tool 'OpenShiftv3.11.0'}"]) {
                        withCredentials([string(credentialsId: 'demo-dev', variable: 'TOKEN')]) {
                            sh 'oc login https://console-openxxxxxxxxxxcom:6443 --token=$TOKEN --insecure-skip-tls-verify'
                        }
                        sh 'oc get projects'
                        sh 'oc project demo-dev'
                        sh 'oc delete -f dc.yml || true'
                        sh 'oc create -f dc.yml'
                        sleep(60)
                    }
                }
            }
        }

        stage('Image Security Test') {
            steps {
                script {
                    sh 'pip install --upgrade "urllib3==1.25.9" anchorecli'
                    def imageLine = 'docker-registry-s2pldemo.pl.s2-eu.xxxxx.com/store:store-develop'
                    writeFile file: 'anchore_images', text: imageLine
                    anchore bailOnFail: false, bailOnPluginFail: false, name: 'anchore_images'
                }
            }
        }

        stage('Functional Tests') {
            agent { label 'slave-0' }
            steps {
                // Run functional tests here
                script {
                    dir('functional-tests') {
                        // Checkout the test source code
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: '*/master']],
                            // Do not clone the history and tags to save time.
                            extensions: [
                                [$class: 'CloneOption', noTags: true, reference: '', shallow: true]
                            ],
                            submoduleCfg: [],
                            userRemoteConfigs: [
                                [ credentialsId: 'fa530f21-e8d3-4b18-9f13-1d1eb2a811e8', url: 'https://xxxsxxxxxxxx.com/gitlab/service-account/store-cucumber.git']
                            ]
                        ])
                        try{
                            // Run  test and proceed to next stages even test fails to import result
                            withMaven(globalMavenSettingsConfig: 'MavenSettings' ,jdk: 'OpenJDK 1.8.181',  maven: 'Maven3', options: [jacocoPublisher(disabled: true), junitPublisher(disabled: true, healthScaleFactor: 1.0)]) {
                                sh 'mvn clean test'
                            }
                        } finally {
                            // Import to XRay
                            step([$class: 'XrayImportBuilder', testPlanKey: 'A00093-4113', endpointName: '/cucumber', importFilePath: 'target/JSON/TestResults.json', serverInstance: 'e6e9d614-8cda--3fd1551f4e0a'])
                            cucumber fileIncludePattern: 'target/JSON/TestResults.json', sortingMethod: 'ALPHABETICAL'
                        }
                    }
                }
            }
        }
        
        stage('OWASP ZAP Security Analysis') {
            steps {
                script {
                    sh '''
                        pip install --upgrade zapcli
                        pip install --upgrade git+https://github.com/Grunny/zap-cli.git
                        #export ZAP_PATH=/root/.ZAP
                        export ZAP_PATH=/root/jenkins/tools/com.cloudbees.jenkins.plugins.customtools.CustomTool/OwaspZap/ZAP_2.9.0
                        zap-cli start -o '-config api.disablekey=true'
                        zap-cli status
                        zap-cli quick-scan -o '-config api.disablekey=true' -s xss,sqli --spider -r http://ip:8090/JSON/httpSessions/view/sites/?apikey=12345
                        zap-cli report -o ZAP_Report.html -f html
                        echo "hello"
                    '''
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '', reportFiles: 'ZAP_Report.html', reportName: 'HTML Report', reportTitles: ''])
                }
            }
        }

        stage('Update Image Stream PROD') {
            steps {
                // Update the image stream of the given image
                script {
                    withEnv(["PATH+OC=${tool 'OpenShiftv3.11.0'}"]) {
                        withCredentials([string(credentialsId: 'demo-prod', variable: 'TOKEN')]) {
                            sh 'oc login https://console-openshift-console.xyz.com:6443 --token=$TOKEN --insecure-skip-tls-verify'
                        }
                        sh 'oc get projects'
                        sh 'oc project demo-prod'
                        sh 'oc import-image s2store'
                    }
                }
            }
        }

        stage('Deployment to PROD') {
            steps {
                script {
                    withEnv(["PATH+OC=${tool 'OpenShiftv3.11.0'}"]) {
                        withCredentials([string(credentialsId: 'xxxxx-demo-prod', variable: 'TOKEN')]) {
                            sh 'oc login https://console-openshift-xxxxxxxxxxxxxxxxxxxx.com:6443 --token=$TOKEN --insecure-skip-tls-verify'
                        }
                        sh 'oc get projects'
                        sh 'oc project demo-prod'
                        sh 'oc delete -f dc.yml || true'
                        sh 'oc create -f dc.yml'
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                def mailRecipients = 'email'
                emailext body: '''${SCRIPT, template="groovy-html.template"}''',
                mimeType: 'text/html',
                subject: 'Mail from CI/CD [Jenkins]',
                to: "${mailRecipients}",
                replyTo: "${mailRecipients}",
                recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'RequesterRecipientProvider'], [$class: 'DevelopersRecipientProvider']]
            }
        }
        cleanup {
            echo 'clean up our workspace'
            // Disable for now to reduce build time
            deleteDir()
        }
    }
}

```
