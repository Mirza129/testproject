#!/usr/bin/env groovy

pipeline {
    agent any

    triggers {
        pollSCM('*/15 * * * *')
    }

    options { disableConcurrentBuilds() }

    stages {
        stage('Cleanup') {
            steps {
                sh './gradlew --no-daemon clean'
            }
        }

        stage('Code analysis') {
            steps {
                sh './gradlew --no-daemon checkstyleMain checkstyleTest findbugsMain findbugsTest pmdMain pmdTest cpdCheck'
            }
            post {
                always {
                    checkstyle pattern: 'build/reports/checkstyle/*.xml', failedTotalAll: '224'
                    dry pattern: 'build/reports/cpd/*.xml'
                    findbugs pattern: 'build/reports/findbugs/*.xml', failedTotalAll: '19'
                    pmd pattern: 'build/reports/pmd/*.xml', failedTotalAll: '0'
                }
            }
        }

        stage('Test') {
            steps {
                sh './gradlew --no-daemon check'
            }
            post {
                always {
                    junit 'build/test-results/test/*.xml'
                    jacoco()
                }
            }
        }

        stage('Build') {
            steps {
                sh './gradlew --no-daemon build'
            }
        }

        stage('Update Docker UAT image') {
            when { branch "1.x.x" }
            steps {
                sh '''
					docker login -u amritendudockerhub -p Passw1rd
                    docker pull amritendudockerhub/person:latest
                    docker build --no-cache -t person .
                    docker tag person:latest amritendudockerhub/person/person:latest
                    docker push amritendudockerhub/person/person:latest
                '''
            }
        }

        stage('Update UAT container') {
            when { branch "1.x.x" }
            steps {
                sh '''
                    docker stop person
                    docker rm person
                    docker run -p 9090:9090 --name person -t -d person
                '''
            }
        }

        stage('Release Docker image') {
            when { buildingTag() }
            steps {
                sh '''
					docker login -u amritendudockerhub -p Passw1rd
					docker pull amritendudockerhub/person:latest
                    docker build --no-cache -t person .
                    docker tag person:latest amritendudockerhub/person/person:${TAG_NAME}
                    docker push amritendudockerhub/person/person:${TAG_NAME}
               '''
            }
        }
    }
}
