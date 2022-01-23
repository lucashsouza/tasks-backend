pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            sh 'mvn clean package -DskipTests=true'
        }
    }
}