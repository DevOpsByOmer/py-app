def COLOR_MAP = [
    SUCCESS: 'good',
    FAILURE: 'danger',
]

pipeline {
    agent any

    environment {
        SONAR_URL = "http://192.168.33.10:9000"
        PYTHON_HOME = tool 'PYTHON'
        APACHE_HOME = "/var/www/html"  // Adjust the path to your Apache installation directory
    }

    tools {
        nodejs 'NODE'
        python 'Python'
        maven 'MVN'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Fetch Code') {
            steps {
                git branch: 'master', url: 'https://github.com/YourUsername/YourPythonApp.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh 'pip install -r requirements.txt'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh 'pytest'
                }
            }
        }

        stage('Sonar Code Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''sonar-scanner \
                        -Dsonar.projectKey=pythonapp \
                        -Dsonar.projectName=pythonapp \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/'''
                }
            }
        }

        stage('Copy to Apache') {
            steps {
                script {
                    // Assuming your Apache server has a directory where you deploy your apps
                    sh "cp -r src/ ${APACHE_HOME}"
                }
            }
        }

        stage('Restart Apache') {
            steps {
                script {
                    // Adjust the command based on your Apache restart command
                    sh "sudo service apache2 restart"
                }
            }
        }
    }

    post {
        always {
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n",
                botUser: false,
                tokenCredentialId: 'slacktoken',
                notifyCommitters: false
        }
    }
}
