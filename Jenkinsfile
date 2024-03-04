//multi-branch

pipeline {

    environment { 
        DOCKERHUB_CREDENTIALS=credentials('dockerhub-credentials')
        MERGE_REPOSITORY='mr'
        MAIN_REPOSITORY='main'
        DEVELOP_REPOSITORY='dev'
    }

    agent {
        // If you want to add agents below, change this to "none".
        node {
            label 'ubuntu-agent'
        }
    }

    stages {
        stage('Pipeline for main branch') {
            // You can add your own remote agent server.

            // agent {
            //     label "linux-main"
            // }
            when {
                branch 'main'
            }
            stages {
                stage('Build Docker Image') {
                    steps {
                        sh 'docker build -t $DOCKERHUB_CREDENTIALS_USR/$MAIN_REPOSITORY:$BUILD_NUMBER .'
                    }
                }

                stage('Build Docker Image') {
                    steps {
                        sh 'docker build -t $DOCKERHUB_CREDENTIALS_USR/$MAIN_REPOSITORY:$BUILD_NUMBER .'
                    }
                }

                stage('Login DockerHub') {
                    steps {
                        sh 'docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW'
                    }
                }

                stage('Deloy Docker Image to DockerHub') {
                    steps {
                        sh 'docker push $DOCKERHUB_CREDENTIALS_USR/$MAIN_REPOSITORY:$BUILD_NUMBER'
                    }
                }
                
                stage('Update Kubernetes Deployment') {
                    steps {
                        script {
                            withCredentials([file(credentialsId: 'kube-config', variable: 'KUBECONFIG-MAIN')]) {
                                sh "kubectl set image deployment/webapp nginx=$DOCKERHUB_CREDENTIALS_USR/$MAIN_REPOSITORY:$BUILD_NUMBER --record"
                            }
                        }
                    }
                }
            }
        }

        stage('Pipeline for dev branch') {
            // You can add your own remote agent server.

            // agent {
            //     label "linux-dev"
            // }
            when {
                branch 'dev'
            }
            stages {
                stage('Build Docker Image') {
                    steps {
                        sh 'docker build -t $DOCKERHUB_CREDENTIALS_USR/$DEVELOP_REPOSITORY:$BUILD_NUMBER .'
                    }
                }

                stage('Build Docker Image') {
                    steps {
                        sh 'docker build -t $DOCKERHUB_CREDENTIALS_USR/$DEVELOP_REPOSITORY:$BUILD_NUMBER .'
                    }
                }

                stage('Login DockerHub') {
                    steps {
                        sh 'docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW'
                    }
                }

                stage('Deloy Docker Image to DockerHub') {
                    steps {
                        sh 'docker push $DOCKERHUB_CREDENTIALS_USR/$DEVELOP_REPOSITORY:$BUILD_NUMBER'
                    }
                }
                
                stage('Update Kubernetes Deployment') {
                    steps {
                        script {
                            withCredentials([file(credentialsId: 'kube-config', variable: 'KUBECONFIG-DEV')]) {
                                sh "kubectl set image deployment/webapp nginx=$DOCKERHUB_CREDENTIALS_USR/$DEVELOP_REPOSITORY:$BUILD_NUMBER --record"
                            }
                        }
                    }
                }
            }
        }

        stage('Pipeline for Merge Request') {
            // You can add your own remote agent server.

            // agent {
            //     label "linux"
            // }

            when {
                branch 'feature-*' ||  branch 'fix-*' 
            }

            stages {
                stage('Generate code style report and save it as artifact') {
                    steps {
                        sh './gradlew checkstyleMain'
                        archiveArtifacts artifacts: 'build/reports/checkstyle/main.html'
                    }
                }

                stage('Test') {
                    steps {
                        sh './gradlew compileJava'
                    }
                }

                stage('Build without test') {
                    steps {
                        sh './gradlew build -x test'
                    }
                }

                stage('Build Docker Image') {
                    steps {
                        sh 'docker build -t $DOCKERHUB_CREDENTIALS_USR/$MERGE_REPOSITORY:$BUILD_NUMBER .'
                    }
                }

                stage('Login DockerHub') {
                    steps {
                        sh 'docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW'
                    }
                }

                stage('Deloy Docker Image to DockerHub') {
                    steps {
                        sh 'docker push $DOCKERHUB_CREDENTIALS_USR/$MERGE_REPOSITORY:$BUILD_NUMBER'
                    }
                }
            }
        }

        post {
            always {
                sh 'docker logout'
            }
        }
    }
}
