pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"

        //input 'Build with maven?'
        withMaven(jdk: 'JDK1.8.0_91', maven: 'Maven 3.3.9', mavenSettingsConfig: '3d93ab17-fa73-416e-abc1-d56831f88d4c') {
        sh '''
                    env
                    echo "PATH = ${PATH}"
                    echo "Build Number = ${BUILD_ID}"
                    mvn -B clean install -Dmaven.test.skip=true
                '''
        //input 'Continue?'
        }
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

            sh 'docker save -o "$WORKSPACE/webgoat.tar" webgoat/webgoat-8.0-${BUILD_ID}'
            nexusPolicyEvaluation(iqStage: 'stage-release', iqApplication: 'webgoat8')
          }
        }
      }
    }
    stage('Publish Container') {
      steps {
        //input 'Push to Nexus Repo?'

        sh '''
                    docker tag webgoat/webgoat-8.0-${BUILD_ID} sonatype.nexus:10351/webgoat/webgoat-8.0-${BUILD_ID}:8.0-${BUILD_ID}
                    docker push sonatype.nexus:10351/webgoat/webgoat-8.0-${BUILD_ID}
                '''
      }
    }
    stage('Create Tag') {
      steps {
        //input 'Create tag in Nexus Repo?'

        createTag nexusInstanceId: 'Nxr3', tagName: "DockerStagingDemoJenkinsfile-Webgoat8-${env.BUILD_ID}"

        associateTag nexusInstanceId: 'Nxr3', search: [[key: 'repository', value: 'DockerDevelopment'], [key: 'name', value: "webgoat/webgoat-8.0-${env.BUILD_ID}"], [key: 'version', value: "8.0-${env.BUILD_ID}"]], tagName: "DockerStagingDemoJenkinsfile-Webgoat8-${env.BUILD_ID}"

        input 'Approve Container for use?'

        moveComponents destination: 'DockerApproved', nexusInstanceId: 'Nxr3', tagName: "DockerStagingDemoJenkinsfile-Webgoat8-${env.BUILD_ID}"

      }
    }
  }
  tools {
    maven 'Maven 3.3.9'
  }
}
