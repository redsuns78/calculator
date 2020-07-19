pipeline {
    agent {
        kubernetes {
            label 'calculator_pod'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:19.03.1
    command:
    - sleep
    args:
    - 99d
    env:
      - name: DOCKER_HOST
        value: tcp://localhost:2375
  - name: docker-daemon
    image: docker:19.03.1-dind
    securityContext:
      privileged: true
    env:
      - name: DOCKER_TLS_CERTDIR
        value: ""
"""
        }
    }
    
    stages {
        stage("Compile") {
            steps {
                sh "./gradlew compileJava"
            }
        }

        stage("Unit test") {
            steps {
                sh "./gradlew test"
            }
        }

        stage("Code coverage") {
            steps {
                sh "./gradlew jacocoTestReport"
                publishHTML (target: [
                                        reportDir: 'build/reports/jacoco/test/html',
                                        reportFiles: 'index.html',
                                        reportName: "JaCoCo Report"
                                ])
                sh "./gradlew jacocoTestCoverageVerification"
            }
        }
        stage('Docker info') {
            steps {
                container('docker') {
                sh 'docker info'
                }
            }
        }
        stage("Static code analysis") {
            steps {
                sh "./gradlew checkstyleMain"
                publishHTML (target: [
                                reportDir: 'build/reports/checkstyle/',
                                reportFiles: 'main.html',
                                reportName: "Checkstyle Report"
                                ])
                        }
                }

        stage("Package") {
            steps {
                sh "./gradlew build"
            }

        }

        //stage("Docker build") {
        //      steps {
        //              sh "docker build -t bistequr55/calculator ."
        //      }
        //}

        stage("Docker build & push") {
            steps {
                echo 'Starting to build docker image'
                script {
                        def customImage = docker.build("bistequr55/calculator")
                        docker.withRegistry('https://registry.hub.docker.com', 'docker-login') {
                            customImage.push()
                        }
                }
            }
        
        stage("Deploy to staging") {
            steps {
               sh "docker run -d --rm -p 8765:8080 --name calculator bistequr55/calculator"
            }
        }

        stage("Acceptance test") {
            steps {
               sleep 60
               //sh "chmod +x acceptance_test.sh && ./acceptance_test.sh"
               sh "./gradlew acceptanceTest -Dcalculator.url=http://localhost:8765"
            }
        }

    }
}
