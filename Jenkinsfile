pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK'
    }

    environment {
        SONARQUBE_ENV = 'SonarQube'
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'jenkins', url: 'https://github.com/pedrose-99/Gestor-Incidencias.git', branch: 'main'
            }
        }

        stage('Build (Back-End)') {
            steps {
                dir('Back-End') {
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage('Test (Back-End)') {
            steps {
                dir('Back-End') {
                    sh 'mvn test'
                }
            }
        }

        stage('Build (Front-End)') {
            agent {
                docker {
                    image 'node:18'
                    args '-u root:root'
                }
            }
            steps {
                dir('Front-End') {
                    sh '''
                        rm -rf node_modules package-lock.json
                        npm install --legacy-peer-deps
                        npm install @angular/cli@8.0.3 --save-dev --legacy-peer-deps
                        npx ng build
                    '''
                }
            }
        }

        stage('Test (Front-End)') {
            agent {
                docker {
                    image 'node:18'
                    args '-u root:root'
                }
            }
            steps {
                dir('Front-End') {
                    sh 'npx ng test --watch=false --browsers=ChromeHeadless'
                }
            }
        }

        stage('SonarQube Analysis (Back-End)') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    dir('Back-End') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('Wait for Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Artifact (Back-End)') {
            steps {
                dir('Back-End') {
                    sh 'mvn package -DskipTests'
                }
            }
        }

        stage('Artifact Upload (Back-End)') {
            steps {
                dir('Back-End') {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: 'nexus:8081',
                        groupId: 'com.gestor',
                        version: '1.0.0',
                        repository: 'maven-releases',
                        credentialsId: 'nexus',
                        artifacts: [
                            [artifactId: 'gestor-incidencias', file: 'target/*.jar', type: 'jar']
                        ]
                    )
                }
            }
        }

        stage('Artifact Upload (Front-End)') {
            steps {
                dir('Front-End/dist') {
                    sh 'echo "Simular subida de artefacto Front-End..."'
                }
            }
        }
    }
}
