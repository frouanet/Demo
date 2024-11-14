def props
def mybd
def mydcs

pipeline {
    agent none 
    tools {
        maven 'Maven'
        jdk 'JDK17'

    }

    stages {
        stage('Build and test') {
            agent any
            steps {
                echo '============  Build'
                sh 'mvn -Dmaven.test.failure.ignore clean package'
            }
            post {
                always { 
                    echo '============  Publish'
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
                success {  
                    echo '============  Archive'  
                    archiveArtifacts '**/target/*.jar'
                    stash name: 'build_result', includes: '**/target/*.jar'
                }
                failure {
                    mail bcc: '', body: 'Ca ne fonctionne pas', cc: '', from: '', replyTo: '', subject: 'package failed', to: 'stageojen@plbformation.com'
                }  
            } 
        }
        stage('Analyse dependance') {            
            parallel {
                stage('Dependances') {
                    agent any
                    steps {
                        echo '=========== Analyse des dépendances du projet'
                    //    sh 'mvn -DskipTests verify'
                        sleep 3
                    }                 
                }
                
                stage('Analyse Sonar') {
                    agent any
                    environment {
                        SONAR_TOKEN = credentials('Sonarqube')
                    }
                    steps {
                        echo '=========== Analyse sonar'
                    //    sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                    }  
                }
                
            }      
        }  
        stage('Deploy') {
            agent any
            when { 
                beforeInput true
                beforeAgent true
                branch 'master'
            } 
            input {
                message 'Dans quel Data Center, voulez-vous déployer l’artefact ?'
                parameters {
                    choice choices: ['Paris ', 'Lille ', 'Lyon'], name: 'DC'
                }
            }
            steps {
                echo "============ Déploiement intégration ${DC}"
                dir("/home/plb/${DC}") {
                    unstash 'build_result'
                } 
            }
        }
        
        stage('Read parms'){
            agent any       
            steps{
                echo "============ Read "
                script{
                    def myjsdata = GetMyDC()
                    mybd = myjsdata['bdir']
                    mydcs = myjsdata['dcs']   
                }  
                
            }
        }
        stage('May I deploy'){
            agent none
            steps{ 
                input (
                    message: 'Pret pour deployer ?',
                    ok: 'deployer'
                )
            } 
        }  
        stage('Deploy 2'){
            agent any
            steps{
                script{
                    for (int i = 0; i < mydcs.size(); ++i) {
                        sh "echo =============== building ${mydcs[i]} to ${mybd}"
                        dir("${mybd}/${mydcs[i]}") {
                            unstash 'build_result'
                        }
                    } 
                } 

            }  
        } 
    }
}

def GetMyDC(){
    def props = readJSON file: './parameters.json'
    def mybuildir = props['builddir'] 
    def mydcs = props['datacenters']
    return ['dcs' : mydcs, 'bdir' : mybuildir] 
}

