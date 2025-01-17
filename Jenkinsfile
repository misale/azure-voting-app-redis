pipeline {
   agent any

   stages {
      stage('Verify Branch') {
         steps {
            echo "$GIT_BRANCH"
         }
      }
      stage('Docker Build') {
         steps {
            sh(script: 'docker images -a')
            sh(script: """
               cd azure-vote/
               docker images -a
               docker build -t jenkins-pipeline .
               docker images -a
               cd ..
            """)
         }
      }
      stage('Trivy - Security Scan') {
         steps {
            sh(script: "trivy jenkins-pipeline")
         }
      }
      stage('Start test app') {
         steps {
            sh(script: """
               docker-compose up -d
               chmod +x ./scripts/test_container.sh
               ./scripts/test_container.sh
            """)
         }
         post {
            success {
               echo "App started successfully :)"
            }
            failure {
               echo "App failed to start :("
            }
         }
      }
      stage('Run Tests') {
         steps {
            sh(script: """
               pytest ./tests/test_sample.py
            """)
         }
      }
      stage('Stop test app') {
         steps {
            sh(script: """
               docker-compose down
            """)
         }
      }
      stage('Push Container') {
         steps {
            echo "Workspace is $WORKSPACE"
            dir("$WORKSPACE/azure-vote") {
               script {
                  docker.withRegistry("https://index.docker.io/v1/", "DockerHub") {
                     def image = docker.build("alabemhar/azure-voting-app:latest")
                     image.push()
                  }
               }
            }
         }
      }
      stage('Test Kubernetes CLI') {
         steps {
            withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://kubernetes.docker.internal:6443/']) {
               sh 'kubectl get pod'
            }
         }
      }
      // stage('Container Scanning') {
      //    parallel {
      //       stage('Run Anchore') {
      //          steps {
      //             pwsh(script: """
      //                Write-Output "blackdentech/jenkins-course" > anchore_images
      //             """)
      //             anchore bailOnFail: false, bailOnPluginFail: false, name: 'anchore_images'
      //          }
      //       }
      //       stage('Run Trivy') {
      //          steps {
      //             sleep(time: 30, unit: 'SECONDS')
      //             // pwsh(script: """
      //             // C:\\Windows\\System32\\wsl.exe -- sudo trivy blackdentech/jenkins-course
      //             // """)
      //          }
      //       }
      //    }
      // }
      // stage('Deploy to QA') {
      //    environment {
      //       ENVIRONMENT = 'qa'
      //    }
      //    steps {
      //       echo "Deploying to ${ENVIRONMENT}"
      //       acsDeploy(
      //          azureCredentialsId: "jenkins_demo",
      //          configFilePaths: "**/*.yaml",
      //          containerService: "${ENVIRONMENT}-demo-cluster | AKS",
      //          resourceGroupName: "${ENVIRONMENT}-demo",
      //          sshCredentialsId: ""
      //       )
      //    }
      // }
      // stage('Approve PROD Deploy') {
      //    when {
      //       branch 'master'
      //    }
      //    options {
      //       timeout(time: 1, unit: 'HOURS') 
      //    }
      //    steps {
      //       input message: "Deploy?"
      //    }
      //    post {
      //       success {
      //          echo "Production Deploy Approved"
      //       }
      //       aborted {
      //          echo "Production Deploy Denied"
      //       }
      //    }
      // }
      // stage('Deploy to PROD') {
      //    when {
      //       branch 'master'
      //    }
      //    environment {
      //       ENVIRONMENT = 'prod'
      //    }
      //    steps {
      //       echo "Deploying to ${ENVIRONMENT}"
      //       acsDeploy(
      //          azureCredentialsId: "jenkins_demo",
      //          configFilePaths: "**/*.yaml",
      //          containerService: "${ENVIRONMENT}-demo-cluster | AKS",
      //          resourceGroupName: "${ENVIRONMENT}-demo",
      //          sshCredentialsId: ""
      //       )
      //    }
      // }
   }
}
