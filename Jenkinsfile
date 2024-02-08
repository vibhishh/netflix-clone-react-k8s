pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'nodejs16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('Clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/vibhishh/netflix-clone-react-k8s.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("Quality Gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Trivy FileSystem Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t netflix ."
                       sh "docker tag netflix mradulsingh25/netflix:latest "
                       sh "docker push mradulsingh25/netflix:latest "
                    }
                }
            }
        }
        stage("Trivy Image Scan"){
            steps{
                sh "trivy image mradulsingh25/netflix:latest > trivy.txt"
            }
        }
        stage('Deploy to kubernetes'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                       sh 'kubectl apply -f deployment.yml'
                       sh 'kubectl apply -f service.yml'
                    }
                }
            }
        }
    }
}
