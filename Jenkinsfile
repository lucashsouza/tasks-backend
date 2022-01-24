pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }   
        }

        stage ('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {   
                    sh "\"${scannerHome}/bin/sonar-scanner\" -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=bfe85633f9d63cf3b1a44b5b10d09f94b002f42b -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=exclusions=**./.mvn/**,**/src/test**,**/model/**,**Application.java"
                }
            }
        }
    }
}