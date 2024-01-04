pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/RAMESHKUMARVERMAGITHUB/docker-spring-boot-java-web-service-example.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=docker-spring-boot-java-web-service \
                    -Dsonar.projectKey=docker-spring-boot-java-web-service'''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                }
            }
        }
        // stage('Install Dependencies') {
        //     steps {
        //         sh "npm install"
        //     }
        // }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                  withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                      sh "docker build -t rameshkumarverma/docker-spring-boot-java-web-service:latest ."
                      // sh "docker tag uber rameshkumarverma/docker-spring-boot-java-web-service:latest "
                      sh "docker push rameshkumarverma/docker-spring-boot-java-web-service:latest"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image rameshkumarverma/docker-spring-boot-java-web-service:latest > trivyimage.txt"
            }
        }
        stage("deploy_docker"){
            steps{
                sh "docker volume create nexus-data"
                sh "docker run -d --name docker-spring-boot-java-web-service -p 8080:8080 rameshkumarverma/docker-spring-boot-java-web-service:latest"
                
            }
        }
        // stage("Deploy"){
        //     steps {
        //         echo "Deploying the container"
        //         sh "docker-compose down && docker-compose build && docker-compose up -d"
                
        //     }
        // }
        // stage('Deploy to kubernets'){
        //     steps{
        //         script{
        //             // dir('K8S') {
        //                 withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
        //                         sh 'kubectl apply -f deployment-service.yml'
        //                         // sh 'kubectl apply -f service.yml'
        //                 }
        //             // }
        //         }
        //     }
        // }
    }
}
