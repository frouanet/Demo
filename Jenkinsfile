pipeline {
    agent any 
    tool name: 'Maven', type: 'maven'
    tool name: 'JDK17', type: 'jdk'

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
        stage('Analyse dependance') {
            parallel {
                stage('Dependances') {
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                        sh 'mvn -DskipTests verify'
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

