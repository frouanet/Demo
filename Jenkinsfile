pipeline {
   agent any 


    stages {
        stage('Build and test') {
            steps {
                echo 'Build'
                sh 'mvn -Dmaven.test.failure.ignore clean package'
            }
            post {
                always { 
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
                success {    
                    archiveArtifacts '**/target/*.jar'
                }
                failure {
                    mail bcc: '', body: 'Ca ne fonctionne pas', cc: '', from: '', replyTo: '', subject: 'package failed', to: 'stageojen@plbformation.com'
                }  
            } 
        }
        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                    }
                    
                }
                 stage('Analyse Sonar') {
                     steps {
                        echo 'Analyse sonar'
                     }
                    
                }
            }
            
        }
            
        stage('Déploiement intégration') {

            steps {
                echo "Déploiement intégration"
                
            }
        }

     }
    
}

