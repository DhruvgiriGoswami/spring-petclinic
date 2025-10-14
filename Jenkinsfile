pipeline {
    agent any

    tools {
        maven 'Maven 3.9.6' // or whatever version you configured in Jenkins
        jdk 'JDK17'         // make sure you have JDK configured
    }

    environment {
        SONARQUBE = credentials('sonar-token') // optional if using credentials manager
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/spring-projects/spring-petclinic.git'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh './mvnw verify sonar:sonar \
                        -Dsonar.projectKey=petclinic \
                        -Dsonar.host.url=http://localhost:9000'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build and analysis successful!'
        }
        failure {
            echo '❌ Build or analysis failed!'
        }
    }
}

stage('Dependency-Track Upload') {
    steps {
        sh '''
        curl -X PUT "http://localhost:8081/api/v1/bom" \
            -H "X-Api-Key: odt_QHc4A3lt_UzhWlAXRvTrEjZRr7zCbRp51eFZYK45h" \
            -H "Content-Type: multipart/form-data" \
            -F "project=af4689e3-a6dc-4c6f-ab60-dddf82a4b226" \
            -F "autoCreate=true" \
            -F "bom=@target/bom.xml"
        '''
    }
}

