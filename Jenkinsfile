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
                git branch: 'main', url: 'https://github.com/Aj7Ay/Hostit.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=hostit \
                    -Dsonar.projectKey=hostit'''
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
                      sh "docker build -t rameshkumarverma/hostit:latest ."
                      // sh "docker tag uber rameshkumarverma/hostit:latest "
                      sh "docker push rameshkumarverma/hostit:latest"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image rameshkumarverma/hostit:latest > trivyimage.txt"
            }
        }
        stage("deploy_docker"){
            steps{
                sh "docker volume create nexus-data"
                // sh "docker run -d --name host -p 8081:8081 rameshkumarverma/hostit:latest"
                sh "docker run -d --name nexus -p 8081:8081 --mount source=nexus-data,target=/nexus-data sonatype/nexus3"
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
