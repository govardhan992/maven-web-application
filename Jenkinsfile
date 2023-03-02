pipeline {
    agent any
    tools {
          maven 'maven-3.9.0' 
    }
    stages {
        stage('check out'){
            steps {
                git 'https://github.com/govardhan992/maven-web-application.git'
            }
        }
        stage('build'){
            steps {
                sh "mvn clean package"
            }
        }
        stage('SonarQube Analysis'){
            steps{
                   withSonarQubeEnv('Sonarqube-8. 9.2') {
                        sh 'mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=maven-web-application \
                        -Dsonar.host.url=http://54.145.159.208:9000   \
                        -Dsonar.login=d6605b0d5c6ae5307454f80b5575e5a1106bc1ed'
                    }
            }
        }
       /* stage("Quality Gate") {
            steps {
                timeout(time: 4, unit: 'MINUTES') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        } */
        stage('build the docker image'){
            steps {
                sh "docker build -t govardhanr992/web:${BUILD_NUMBER} ."
            }
        }
        stage("push docker image"){
            steps {
                withCredentials([string(credentialsId: 'dockerhub_psw', variable: 'dockerhub_psw')]) {
                
               sh "docker login -u govardhanr992 -p ${dockerhub_psw}"
     }
                sh "docker push govardhanr992/web:${BUILD_NUMBER}"
            }
        }
        stage("deploy_app"){
            steps {
              script {

                 def USER_INPUT = input(
                    message: 'User input required - Do you want to proceed?',
                    parameters: [
                            [$class: 'ChoiceParameterDefinition',
                             choices: ['no','yes'].join('\n'),
                             name: 'input',
                             description: 'Menu - select box option']
                    ])
                    echo "The answer is: ${USER_INPUT}"
                    if( "${USER_INPUT}" == "yes"){
                sshagent(['deploy_container']) {
                 
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.88.104 docker pull govardhanr992/web:${BUILD_NUMBER}'
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.88.104 docker rm -f webserver || true'
                 sh 'ssh -o StrictHostKeyChecking=no ec2-user@172.31.88.104 docker run -d -p 8090:8080 --name webserver govardhanr992/nginx:${BUILD_NUMBER}'
                }
                   
            }
                else {
                   echo "Skipping deployment"
                }
         }
            }
        }
    }
    }
}
