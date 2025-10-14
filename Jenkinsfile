pipeline {
    agent any

    tools {
        maven 'Maven 3.9.9'   // Make sure this is defined in Jenkins Global Tool Config
        jdk 'JDK17'           // Same name as your JDK tool in Jenkins
    }

    environment {
        SONARQUBE = credentials('sonar-token') // Jenkins credential ID for SonarQube token
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DhruvgiriGoswami/spring-petclinic'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') { // Must match your Jenkins SonarQube server name
                    sh """
                    ./mvnw verify sonar:sonar \
                        -Dsonar.projectKey=petclinic \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.token=${SONARQUBE}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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
    }

    post {
        success {
            echo '✅ Build, analysis, and upload successful!'
        }
        failure {
            echo '❌ Build or analysis failed!'
        }
    }
}

