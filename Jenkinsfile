pipeline {
   agent any 
    tools {
        jdk 'JDK17'
        maven 'MAVEN3'
    }


    stages {
        stage('Compile et tests') {
            steps {
                echo 'Unit test et packaging'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
                dir('application/target') {
                    stash includes: '*.jar', name: 'app'
                }
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'                }
                success {
                    archiveArtifacts artifacts: '**/target/*.jar', followSymlinks: false
                }
                unsuccessful {
                    mail bcc: '', body: 'Please review', cc: '', from: '', replyTo: '', subject: 'Build failed', to: 'david.thibau@sparks.com'
                }
            }
        }
        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
                    agent any
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                        sh 'mvn -DskipTests verify'
                    }
                    
                }
                 stage('Analyse Sonar') {
                    agent any
                    environment {
                        SONAR_TOKEN = credentials('SONAR_TOKEN')
                    }
                     steps {
                        echo 'Analyse sonar'
                        sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                     }
                    
                }
            }
            
        }
            
        stage('Déploiement intégration') {
            agent any
            input {
                message 'Vers quel data center voulez vous déployer ?'
                ok 'Déployer'
                parameters {
                    choice choices: ['PARIS', 'NANCY', 'STRASBOURG'], name: 'DATACENTER'
                }
            }
            steps {
                echo "Déploiement intégration vers $DATACENTER"
                unstash 'app'
                sh "cp *.jar /home/dthibau/Formations/Jenkins/MyWork/Serveurs/${DATACENTER}.jar"
            }
        }

     }
    
}

