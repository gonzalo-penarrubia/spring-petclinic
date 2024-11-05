#!/usr/bin/env groovy
pipeline {
    agent { label 'pod-default' }
    stages {
        stage('Check Environment') {
            steps {
                sh 'java -version'
                container('maven') {
                    sh '''
                        java -version
                        mvn -version
                        pwd
                    '''
                }
            }
        }
        stage('Build') {
            steps {
                container('maven') {
                    println '01# Stage - Build'
                    println '(develop y main):  Build a jar file.'
                    sh './mvnw package -Dmaven.test.skip=true'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
        stage('Unit Tests') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                container('maven') {
                    println '03# Stage - Unit Tests'
                    println '(develop y main): Launch unit tests.'
                    sh '''
                        mvn test
                    '''
                    junit '**/target/surefire-reports/*.xml'
        }
    }
}
    }
}
    }
}
