pipeline {
    agent any

    environment {
        // SonarQube Environment Variables
        SONAR_HOST_URL = 'http://localhost:9000' // Replace with your SonarQube server URL
        SONAR_AUTH_TOKEN = 'all_project_token'      // Replace with your SonarQube authentication token
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code from GitHub...'
                checkout scm // Automatically checks out the repository configured in Jenkins
            }
        }

        stage('Build Project') {
            steps {
                echo 'Building the Maven project...'
                sh 'mvn clean install'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running test cases...'
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('sonarqube server') { // 'SonarQube' is the name of your SonarQube server in Jenkins
                    sh 'mvn sonar:sonar -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_AUTH_TOKEN'
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                script {
                    echo 'Checking SonarQube Quality Gate...'
                    timeout(time: 5, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution finished.'
        }
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
