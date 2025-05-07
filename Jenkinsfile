pipeline {
    agent any

    environment {
        MAVEN_HOME = tool 'Maven'
        NODE_HOME = tool 'NodeJS'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'jenkins', url: 'https://github.com/pedrose-99/Gestor-Incidencias.git']])
            }
        }

        stage('Install NodeJS and Build (Front-End)') {
            steps {
                dir('Front-End') {
                    sh '''
                        curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
                        apt-get install -y nodejs
                        node -v
                        npm -v

                        npm install -g @angular/cli@8.0.3 --legacy-peer-deps
                        npm install --legacy-peer-deps
                        npm run build
                    '''
                }
            }
}
        stage('Build (Back-End)') {
            steps {
                dir('Back-End') {
                    sh "${MAVEN_HOME}/bin/mvn compile"
                }
            }
        }

        stage('Test (Back-End)') {
            steps {
                dir('Back-End') {
                    sh "${MAVEN_HOME}/bin/mvn test"
                }
            }
        }

        stage('Build (Front-End)') {
            steps {
                dir('Front-End') {
                    withEnv(["PATH+NODE=${NODE_HOME}/bin"]) {
                        sh 'rm -rf node_modules package-lock.json'
                        sh 'npm install --legacy-peer-deps'
                        sh 'npm run build'
                    }
                }
            }
        }

        stage('Test (Front-End)') {
            steps {
                dir('Front-End') {
                    sh 'npm run test'
                }
            }
        }

        stage('SonarQube Analysis (Back-End)') {
            steps {
                dir('Back-End') {
                    withSonarQubeEnv('sonarQube') {
                        withCredentials([string(credentialsId: 'jenkins-sq1', variable: 'SONAR_TOKEN')]) {
                            sh "${MAVEN_HOME}/bin/mvn sonar:sonar " +
                               "-Dsonar.projectKey=Gestor-Incidencias " +
                               "-Dsonar.projectName=Gestor-Incidencias " +
                               "-Dsonar.host.url=https://8eea-2a02-9140-4f80-3500-ea23-8799-ce08-99cd.ngrok-free.app " +
                               "-Dsonar.login=${SONAR_TOKEN}"
                        }
                    }
                }
            }
        }

        stage('Wait for Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Artifact (Back-End)') {
            steps {
                dir('Back-End') {
                    sh "${MAVEN_HOME}/bin/mvn package"
                }
            }
        }

        stage('Artifact (Front-End)') {
            steps {
                dir('Front-End/dist') {
                    sh 'tar -czvf frontend-dist.tar.gz *'
                }
            }
        }

        stage('Artifact Upload (Back-End)') {
            steps {
                dir('Back-End') {
                    nexusArtifactUploader artifacts: [[
                        artifactId: 'issuetracking',
                        classifier: '',
                        file: 'target/issuetracking-0.0.1-SNAPSHOT.jar',
                        type: 'jar'
                    ]], credentialsId: 'Nexus',
                    groupId: 'com.ismail',
                    nexusUrl: 'a316-2a02-9140-4f80-3500-ea23-8799-ce08-99cd.ngrok-free.app',
                    nexusVersion: 'nexus3',
                    protocol: 'https',
                    repository: 'gestor-incidencias',
                    version: '0.0.1-SNAPSHOT'
                }
            }
        }

        stage('Artifact Upload (Front-End)') {
            steps {
                dir('Front-End/dist') {
                    nexusArtifactUploader artifacts: [[
                        artifactId: 'frontend',
                        classifier: '',
                        file: 'frontend-dist.tar.gz',
                        type: 'tar.gz'
                    ]], credentialsId: 'Nexus',
                    groupId: 'com.ismail.frontend',
                    nexusUrl: 'a316-2a02-9140-4f80-3500-ea23-8799-ce08-99cd.ngrok-free.app',
                    nexusVersion: 'nexus3',
                    protocol: 'https',
                    repository: 'gestor-incidencias',
                    version: '0.0.1-SNAPSHOT'
                }
            }
        }
    }
}
