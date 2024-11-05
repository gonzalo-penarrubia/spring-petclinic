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

        stage('Publish Artifact') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                container('maven') {
                    println '04# Stage - Deploy Artifact'
                    println '(develop y main): Deploy artifact to repository.'
                    sh '''
                        mvn -e deploy:deploy-file \
                            -Durl=http://nexus-service:8081/repository/maven-snapshots \
                            -DgroupId=local.moradores \
                            -DartifactId=spring-petclinic \
                            -Dversion=3.3.0-SNAPSHOT \
                            -Dpackaging=jar \
                            -Dfile=target/spring-petclinic-3.3.0-SNAPSHOT.jar
                    '''
                }
            }
        }
    }
}
    }
}
    }
}
