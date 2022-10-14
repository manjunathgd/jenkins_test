pipeline {
    agent any
    
    tools {
        // Install the Maven version configured as "M3" and add it to the  path.
        maven "MVN3"
        jdk "jdk8"
    }

    stages {
        stage('pullscm') {
            steps {
                git credentialsId: 'GitHub', url: 'git@github.com:manjunathgd/jenkins_test.git'
            }
        }
        
                
        stage('Build') {
            steps {
                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true -f api-gateway clean package"

                // To run Maven on a Windows agent, use
                // bat "mvn -Dmaven.test.failure.ignore=true -f api-gateway clean package"
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    junit 'api-gateway/target/surefire-reports/*.xml'
                    archiveArtifacts 'api-gateway/target/*.jar'
                }
            }
        }
        
    stage('pulltestingcode') {
      steps {
        git branch: 'main', credentialsId: 'GitHub', url: 'git@github.com:manjunathgd/functional-testing.git'
      }
    }
    stage('execute test') {
      steps {
        sh "mvn clean test"
      }
         post {
              success {
                  publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'TestReport', reportFiles: 'TestReport.html', reportName: 'FunctionalTestReport', reportTitles: '', useWrapperFileDirectly: true])
                  sh 'rm -rf *'
                  withCredentials([sshUserPrivateKey(credentialsId: 'GitHub', keyFileVariable: 'SSH_KEY')]) {
                      sh 'echo ssh -i $SSH_KEY -l git -o StrictHostKeyChecking=no \\"\\$@\\" > local_ssh.sh'
                      sh 'chmod +x local_ssh.sh'
                      withEnv(['GIT_SSH=/var/lib/jenkins/workspace/pipeline/local_ssh.sh']) {
                          sh 'git clone git@github.com:manjunathgd/jenkins_test.git'
                          sh '''cd jenkins_test
                          echo test>deploy.txt
                          git add .
                          git commit -m "merging master to qa on sucesfull build"
                          git checkout -b qa
                          git pull origin qa
                          git push origin qa'''
                      }
                  }   
              }
         }
    }
        
  }
}

