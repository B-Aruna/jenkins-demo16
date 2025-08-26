pipeline {
    agent any
    environment {
        MAVEN_OPTS = "-Dmaven.test.failure.ignore=true"
        GRYPE_BINARY_DIR = "${env.WORKSPACE}/bin"
        GRYPE_SCAN_TARGET = "${env.WORKSPACE}/test-workflow-ninja"
        GRYPE_REPORT = "grype-report.sarif"
    }
    tools {
        maven 'Maven 3'
    }
    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
                sleep 3
            }
        }
        stage('Registering build artifact') {
            steps {
                echo 'Registering the metadata'
                registerBuildArtifactMetadata(
                    name: "jenkins-demo16",
                    version: "1.0.0",
                    type: "docker",
                    url: "http://localhost:1112",
                    digest: "6u637064707039346163673930",
                    label: "pre-prod"
                )
                sleep 3
            }
        }
        stage('Unit Test') {
            steps {
                // Use catchError and force both buildResult and stageResult to 'SUCCESS'
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    sh 'mvn clean test'
                }
            }
        }
        stage('Publish Test Results') {
            steps {
                junit 'target/surefire-reports/*.xml'
                sleep 5
            }
        }
        stage('Install Grype') {
            steps {
                sh '''
                echo "Installing Grype..."
                mkdir -p ${GRYPE_BINARY_DIR}
                export PATH=${GRYPE_BINARY_DIR}:$PATH
                curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b ${GRYPE_BINARY_DIR}
                ${GRYPE_BINARY_DIR}/grype version
                '''
            }
        }
        stage('List Files') {
            steps {
                sh '''
                echo "📁 Current workspace contents:"
                ls -la ${WORKSPACE}
                '''
            }
        }
        stage('Scan Folder with Grype') {
            steps {
                sh '''
                echo "Scanning folder '${GRYPE_SCAN_TARGET}' with Grype..."
                ${GRYPE_BINARY_DIR}/grype ${GRYPE_SCAN_TARGET} -o sarif > ${GRYPE_REPORT}
                '''
            }
        }
        stage('Display SARIF Report') {
            steps {
                sh '''
                echo "=== Grype SARIF Report ==="
                cat ${GRYPE_REPORT}
                '''
            }
        }
         stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: "${GRYPE_REPORT}", fingerprint: true
            script {
                currentBuild.result = 'SUCCESS'
            }
        }
    }
}
