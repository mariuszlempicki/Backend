def imageName="mariuszlempicki/panda_backend"
def dockerRegistry=""
def registryCredentials="dockerhub"
def dockerTag=""


pipeline {
    agent {
        label "agent"
    }
    
    environment {
        scannerHome = tool 'SonarQube'
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Start Backend Pipe'
            }
            
        }
        stage('Clone Backend repo') {
            steps {
                git branch: 'main', url: 'https://github.com/mariuszlempicki/Backend.git'
            }
        }
        
        stage('Files') {
            steps {
                sh "ls -la"
            }
        }  
        
        stage('Run tests') {
            steps {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytestreport.xml"
            }
        }   
        
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build application image') {
            steps {
                script {
                  // Prepare basic image for application
                  dockerTag = "RC-${env.BUILD_ID}"
                  applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
        }
        
        stage('Pushing image to Artifactory') {
            steps {
                script {
                  docker.withRegistry("$dockerRegistry", "$registryCredentials"){
                      applicationImage.push();
                      applicationImage.push("latest");
                  }
                }
            }
        }
    }
    
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }
}
