pipeline {
    agent any
    tools {
        jdk "java17"
        nodejs "node16"
    }
    environment {
        SCANNER_HOME=tool"mysonarqube"
    }
    stages {
        stage ("fetch code from github"){
            steps{
                git branch: 'main', url: 'https://github.com/madhavareddy97056/zomato_code.git'
            }
        }
        stage ("sonar scan the source code"){
            steps{
                withSonarQubeEnv("mysonarqube") {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=madhava_project \
                         -Dsonar.projectKey=zomato'''
                }
            }
        }
        stage ("Quality gates verification") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube_creds'
                }
            }
        }
        stage ("installing all dependencies through npm"){
            steps {
                sh "npm install"
            }
        }
        stage ("OWASP scan for dependencies"){
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'owasp-dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage ("Create Docker image"){
            steps {
                sh "docker build -t madhavareddy97056/project_2:$image_name ."
            }
        }
        stage ("scanning the docker image") {
            steps {
                sh "trivy image madhavareddy97056/project_2:$image_name"
            }
        }
        stage ("Push Docker image"){
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker_creds') {
                        sh "docker push madhavareddy97056/project_2:$image_name"
                    }
                }
            }
        }
        stage ("deploy container"){
            steps {
                sh "docker run -d --name $container_name -p $host_port:3000 madhavareddy97056/project_2:$image_name"
            }
        }
    }
}
