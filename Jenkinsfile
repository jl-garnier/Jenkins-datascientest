// at the pipeline and stage level
pipeline {
    agent any // means any agent
    parameters {
        string(name: 'user', defaultValue: 'John', description: 'A user that triggers the pipeline')
    }
     // environment variables at pipeline level
    environment {
         DOCKER_ID = 'garnierjl'
         DOCKER_IMAGE = 'datascientestapi'
         DOCKER_TAG = "v.${BUILD_ID}.0" 
         DOCKERHUB_CREDENTIALS = credentials('docker_jenkins')  // variable secret

    }
    // tools {
    //     python 'python3'
    // }
    stages {

        stage('Building') {

            steps {
                echo 'Install dependencies'
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Testing') {

            steps {
                // script { 
                //     echo 'Execute units tests'
                //     sh 'python3.8 -m unittest'
                // }
                withPythonEnv('python3.8') {
                    sh '''
                    python3.8 -m pip install flask
                    python3.8 -m unittest
                    '''
                }
            }

        }

        stage('Deploying') {

            steps {
                echo 'Deploy application'
                script {
                    sh '''
                    docker rm -f jenkins
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                    docker run -d -p 8000:8000 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                    echo 'Build Image Completed'
                }
            }
        }


        stage('User Acceptance') {
            steps {
                script {
                    def returnValue = input(message: "Approve push on main?", ok: "Yes")
                    echo "User: ${params.user} requested to push the code to the main branch"
                }
            }
        }

        stage('Pushing and Merging ') {
            
            parallel {
                stage('Pushing Image') {
                    // set environment variables at stage level using docker credentials
                    environment {
                        DOCKERHUB_CREDENTIALS_USR = credentials('docker_jenkins')  // variable secret
                        DOCKERHUB_CREDENTIALS_PSW =  credentials('DOCKER_HUB_PASS')  // variable secret

                    }
                    steps {
                        sh 'echo $DOCKERHUB_CREDENTIALS'
                        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                        echo 'Login Completed'
                        sh 'docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG'
                        echo 'Push Image Completed'
                      }
                  }
                  stage('Merging') {
                        steps {
                            echo 'Merging done'
                    }
                }
            }
        }
        // stage('Deploy stage') {
        //     when {
        //         branch 'master'
        //     }
        //     steps {
        //         echo 'Deploy master to stage'
        //         // Add steps for deployment
        //     }
        // }
        // stage('SonarQube analysis') {
        //     steps {
        //         script {
        //         // requires SonarQube Scanner 2.8+
        //         scannerHome = tool 'SonarQube Scanner 2.8'
        //         }
        //         withSonarQubeEnv('SonarQube Scanner') {
        //             sh "${scannerHome}/bin/sonar-scanner"
        //         }
        //     }
        // }
    
    }
    post {
        always {
            echo "Pipeline finished disconnect form dockerhub"
            sh "docker logout"
        }
    }
}