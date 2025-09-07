pipeline {
    agent any

    tools {
        maven 'MVN_HOME'     // From Jenkins Global Tool Configuration
    }

    environment {
        SCANNER_HOME = tool 'sonarscanner'   // SonarQube Scanner tool
        NEXUS_REPO_URL = "http://52.91.43.63:8081/repository/simple-app/"
        SONARQUBE_SERVER = "SonarQube"       // From SonarQube Jenkins config
    }

    stages {

        stage('Checkout SCM') {
            steps {
                git branch: 'feature-1.1', url: 'https://github.com/prakash6333/sabear_simplecutomerapp.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh """
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Ncodeit \
                        -Dsonar.projectName=Ncodeit \
                        -Dsonar.projectVersion=${BUILD_NUMBER} \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                sh """
                    mvn deploy -DskipTests \
                        -DaltDeploymentRepository=simple-app::default::${NEXUS_REPO_URL} \
                        --settings /var/lib/jenkins/.m2/settings.xml
                """
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat_credentials', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                        WAR_FILE=$(ls target/*.war | head -n 1)
                        curl -u $TOMCAT_USER:$TOMCAT_PASS \
                             -T $WAR_FILE \
                             "http://107.21.137.31:8080/manager/text/deploy?path=/hiring&update=true"
                    '''
                }
            }
        }

        stage('Slack Notification') {
            steps {
                slackSend(
                    channel: '#jenkins-integration',
                    color: 'good',
                    message: "✅ Build ${env.JOB_NAME} #${env.BUILD_NUMBER} finished successfully",
                    tokenCredentialId: 'slack_integration'
                )
            }
        }
    }

    post {
        failure {
            slackSend(
                channel: '#jenkins-integration',
                color: 'danger',
                message: "❌ Build ${env.JOB_NAME} #${env.BUILD_NUMBER} failed!",
                tokenCredentialId: 'slack_integration'
            )
        }
        always {
            echo "Pipeline finished"
        }
    }
}
