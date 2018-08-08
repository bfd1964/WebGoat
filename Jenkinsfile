pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"

        //input 'Build with maven?'

        sh '''
                    env
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    echo "Build Number = ${BUILD_ID}"
                    mvn -B clean install -Dmaven.test.skip=true
                '''
        //input 'Continue?'
      }
    }
    stage('Scan App - Build Container') {
      parallel {
        stage('IQ-BOM') {
          steps {
            //input 'Scan with IQ at Build?'

            nexusPolicyEvaluation(iqApplication: 'webgoat8', iqStage: 'build', iqScanPatterns: [[scanPattern: '']])
          }
        }
        stage('Static Analysis') {
          steps {
            echo '...run SonarQube or other SAST tools here'
          }
        }
        stage('Build Container') {
          steps {
    
            //input 'Build with docker?'

            sh '''cd webgoat-server
                  docker build -t webgoat/webgoat-8.0-${BUILD_ID} .
                    '''
          }
        }
      }
    }
    stage('Test Container') {
      parallel {
        stage('Test Container') {
          steps {
            catchError() {
              echo 'Test Container here'
            }

          }
        }
        stage('IQ-Scan Container') {
          steps {
            //input 'Scan with IQ at Stage-Release?'

            sh 'docker save -o $WORKSPACE/webgoat.tar webgoat/webgoat-8.0-${BUILD_ID}'
            nexusPolicyEvaluation(iqStage: 'stage-release', iqApplication: 'webgoat8')
          }
        }
      }
    }
    stage('Publish Container') {
      when {
        branch 'develop'
      }
      steps {
        //input 'Push to Nexus Repo?'

        sh '''
                    docker tag webgoat/webgoat-8.0-${BUILD_ID} local-mike:19447/webgoat/webgoat-8.0-${BUILD_ID}:8.0-${BUILD_ID}
                    docker push local-mike:19447/webgoat/webgoat-8.0-${BUILD_ID}
                '''
      }
    }
    stage('Create Tag') {
      when {
        branch 'develop'
      }
      steps {
        //input 'Create tag in Nexus Repo?'

        createTag nexusInstanceId: 'nexus3-demo', tagName: "DockerStagingDemoJenkinsfile-Webgoat8-${env.BUILD_ID}"

        associateTag nexusInstanceId: 'nexus3-demo', search: [[key: 'repository', value: 'docker-hosted-beta'], [key: 'name', value: "webgoat/webgoat-8.0-${env.BUILD_ID}"], [key: 'version', value: "8.0-${env.BUILD_ID}"]], tagName: "DockerStagingDemoJenkinsfile-Webgoat8-${env.BUILD_ID}"

        input 'Move Image out of Beta?'

        moveComponents destination: 'docker-hosted', nexusInstanceId: 'nexus3-demo', tagName: "DockerStagingDemoJenkinsfile-Webgoat8-${env.BUILD_ID}"



      }
    }
  }
  tools {
    maven 'Maven 3.5.3 - Local Install'
  }
}
