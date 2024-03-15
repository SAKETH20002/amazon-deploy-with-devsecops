pipeline {
    agent any
    tools {
         jdk 'JAVA_HOME' 
         nodejs 'NODEJS'
    }

    stages {
        stage('namstey') {
            steps {
               cleanWs()
            }
        }
        stage('git') {
            steps {
               git branch: 'main', url: 'https://github.com/Aj7Ay/Amazon-FE.git' 
            }
        }
        stage('Sonarqube') {
            
            environment {
                sonnarscanner = tool 'SonarScanner4'
            }
            steps{
             script{
                   withSonarQubeEnv('sonarqube') { 
                   sh ''' ${sonnarscanner}/bin/sonar-scanner -Dsonar.projectName=Amazon-deploy \
                    -Dsonar.projectKey=Amazon-deploy \
                     
                    '''
                   }
                } 
               
            }
        }
        stage ("quality gate"){
                   steps{
                       script{
                          waitForQualityGate abortPipeline: false, credentialsId: 'Sonarqube'
                      }
                  }
        }
        
        stage("npm dependencies"){
            steps{
                sh 'npm install'
            }
        }
        
        stage ("owasp scan"){
            steps{
                 dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
                
            
        }
        
         stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('docker push'){
            steps{
                script{
                    withDockerRegistry(credentialsId: "dockerhub" , toolName: "DOCKER"){
                        sh "docker build -t theamazon ."
                        sh "docker tag theamazon sakethmutyala/theamazon:latest "
                        sh "docker push sakethmutyala/theamazon:latest "
                    } 
                }
            }
        }
        stage("trivy scan image"){
            steps{
                sh "trivy image sakethmutyala/theamazon:latest > trivyimage.txt"
            }
        }
        
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name amazon -p 3000:3000 sakethmutyala/theamazon:latest'
            }
        }
    
    }
}
