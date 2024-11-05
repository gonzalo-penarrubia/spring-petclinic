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
            }
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
        stage('Build & Publish Container Image') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                container('kaniko') {
                    println '05# Stage - Build & Publish Container Image'
                    println '(develop y main): Build container image with Kaniko & Publish to container registry.'
                    sh '''
                        /kaniko/executor \
                        --context pwd \
                        --insecure \
                        --dockerfile Dockerfile \
                        --destination=nexus-service:8082/repository/docker/spring-petclinic:3.3.0-SNAPSHOT \
                        --destination=nexus-service:8082/repository/docker/spring-petclinic:latest \
                        --build-arg VERSION=3.3.0-SNAPSHOT.jar
                    '''
                }
            }
            stage('Deploy petclinic') {
                when {
                    anyOf {
                        branch 'main'
                        branch 'develop'
                    }
                }
                steps {
                    println '06# Stage - Deploy petclinic'
                    println '(develop y main): Deploy petclinic app to MicroK8s.'
                    sh '''
                        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                        chmod +x kubectl
                        mkdir -p ~/.local/bin
                        #mv ./kubectl ~/.local/bin/kubectl
                        
                        ./kubectl version --client
                        ./kubectl get all

                        ./kubectl create deployment petclinic --image http://10.152.183.252/:8082/repository/docker/spring-petclinic:latest || \
                        echo 'Deployment petclinic already exists, creating service...'
                        ./kubectl expose deployment petclinic --port 8080 --target-port 8080 --selector app=petclinic --type ClusterIP --name petclinic

                        ./kubectl get all
                    '''
                }
            }
        }
    }
}
