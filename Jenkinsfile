pipeline {
    agent any 
    tools {
        maven 'Maven'
        jdk 'JDK17'
    } 

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
            environment {
                SONAR_TOKEN = credentials('Sonarqube')
            }
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
                    sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                    }  
                }
            }      
        }
            
        stage('Last question') {
            input {
                message 'Dans quel Data Center, voulez-vous déployer l’artefact ?'
                parameters {
                    choice choices: ['Paris ', 'Lille ', 'Lyon'], name: 'DC'
                }
            }
            steps {
                echo "Déploiement intégration ${DC}"
                
            }
        }

     }
    
}

