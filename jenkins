pipeline {
    agent any

    environment {
        APP_NAME = "NumberGuessGame-1.0-SNAPSHOT"
        TOMCAT_HOST = "54.147.11.33"
        TOMCAT_USER = "admin"
        TOMCAT_PASS = "YourSecurePassword"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', 
                    credentialsId: '446646e8-c9e9-4505-ab97-372912e4b91a', 
                    url: 'https://github.com/otejuoso/numberguessgame.git'
            }
        }

        stage('Build & Package') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Sonarqube integration') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh """
                if [ ! -f target/${APP_NAME}.war ]; then
                    echo 'ERROR: WAR file not found! Build failed.' && exit 1
                fi

                curl --upload-file target/${APP_NAME}.war \
                     -u ${TOMCAT_USER}:${TOMCAT_PASS} \
                     "http://${TOMCAT_HOST}:8080/manager/text/deploy?path=/${APP_NAME}&update=true"
                """
            }
        }

        stage('Post-Deployment Check') {
            steps {
                script {
                    def appURL = "http://${TOMCAT_HOST}:8080/${APP_NAME}/"
                    def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' ${appURL}", returnStdout: true).trim()

                    if (response == '200') {
                        echo "Deployment successful! Click to Open: ${appURL}"
                    } else {
                        error("Deployment failed! HTTP Status: ${response}")
                    }
                }
            }
        }
    }

    post {
        failure {
            mail to: 'wbrymo@gmail.com',
                 subject: "Build Failed: ${APP_NAME}",
                 body: "Check Jenkins logs for details.\n\nApp URL: http://${TOMCAT_HOST}:8080/${APP_NAME}/"
        }
    }
}
