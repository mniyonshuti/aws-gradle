pipeline {

    agent any

    environment {
        registry = "mniyonshuti/test2"
        registryCredential = 'niyo-docker'
        dockerImage = ''
    }
    stages{
        stage('SCM Checkout'){
            steps {
                git 'https://github.com/mniyonshuti/aws-gradle'
            }
        }
        stage(' Gradle Clean'){
            steps{
                sh './gradleW clean'
            }
        }
        stage(' Gradle build'){
            steps{
                 sh './gradleW build'
            }
        }
        stage('Gradle Compile'){
            steps  {
                sh './gradleW compileJava'
            }
        }
        stage('Gradle Test'){
            steps {
                sh './gradlew test'
            }
        }
        stage('Gradle Package'){
            steps {
                sh './gradlew  bootJar'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }

        stage('Deploy Docker Image') {
            steps {
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Cleaning up') {
            steps {
                sh "docker rmi $registry:$BUILD_NUMBER"
            }
        }

        stage('Deploy to AWS') {
            environment {
               DOCKER_HUB_LOGIN = credentials('niyo-docker')
            }
            steps {
                withAWS(credentials: 'niyo-aws-credential', region: env.REGION) {
                  sh './gradlew awsCfnMigrateStack awsCfnWaitStackComplete -PsubnetId=$SUBNET_ID -PdockerHubUsername=$DOCKER_HUB_LOGIN_USR -Pregion=$REGION'
                }
            }
        }
    }
}
