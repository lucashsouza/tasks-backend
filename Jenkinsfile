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

        stage ('Quality Gate') {
            steps {
                sleep(10)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8090/')], contextPath: 'tasks-backend', war:'target/tasks-backend.war'
            }
        }

        stage ('API Test') {
            steps {
                dir('api-test') {
                    git branch: 'main', url: 'https://github.com/lucashsouza/tasks-api-test'
                    sh 'mvn test'
                }
            }
        }

        stage ('Deploy Frontend') {
            steps {
                dir('frontend') {
                    git branch: 'master', url: 'https://github.com/lucashsouza/tasks-frontend'
                    sh 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8090/')], contextPath: 'tasks', war:'target/tasks.war'
                }
            }
        }

        stage ('Functional Test') {
            steps {
                dir('functional-test') {
                    git branch: 'main', url: 'https://github.com/lucashsouza/tasks-functional-test'
                    sh 'mvn test'
                }
            }
        }

        stage ('Deploy Prod') {
            steps {
                sh 'docker-compose build'
                sh 'docker-compose up -d'
            }
        }

        stage ('Health Check') {
            steps {
                sleep(10)
                dir('functional-test') {
                    sh 'mvn verify -Dskip.surefire.tests'
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, functional-test/target/surefire-reports/*.xml, functional-test/target/failsafe-reports/*.xml'
        }
    }
}