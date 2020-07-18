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
    }
}
