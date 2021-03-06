pipeline {
  agent any
  stages {
    stage('Git Checkout') {
      parallel {
        stage('Git Checkout') {
          steps {
            git(url: 'https://github.com/tsheth/docker-exploit-demo.git', credentialsId: 'git-creds')
          }
        }
        stage('Static code analysis') {
          steps {
            sh '''echo "static code analysis for github code"
sleep 12'''
          }
        }
      }
    }
    stage('Build') {
      parallel {
        stage('Build Cluster service') {
          steps {
            sh '''rm -rf DockerbaseDSSC
git clone https://github.com/tsheth/DockerbaseDSSC.git
cd DockerbaseDSSC
docker build -t cluster-service:latest .'''
          }
        }
        stage('Provision VMware Environment') {
          steps {
            sh '''/usr/local/bin/terraform init
/usr/local/bin/terraform apply --auto-approve'''
          }
        }
        stage('Build Java Location Service') {
          steps {
            sh '''rm -rf struts-app
git clone https://github.com/tsheth/struts-app.git
cd struts-app
docker build -t location-service:latest .'''
          }
        }
      }
    }
    stage('Test') {
      parallel {
        stage('Performance Test') {
          steps {
            sh '''echo "local testing results"
sleep 20'''
          }
        }
        stage('Docker image scanning') {
          steps {
            sh '''docker run -v /var/run/docker.sock:/var/run/docker.sock deepsecurity/smartcheck-scan-action --image-name cluster-service:latest --smartcheck-host="dssc.bryceindustries.net" --smartcheck-user="administrator" --smartcheck-password="Trend@123" --insecure-skip-tls-verify --insecure-skip-registry-tls-verify --preregistry-scan --preregistry-user admin --preregistry-password Trend@123 --findings-threshold \'{"malware": 100, "vulnerabilities": { "defcon1": 100, "critical": 100, "high": 100 }, "contents": { "defcon1": 100, "critical": 100, "high": 100 }, "checklists": { "defcon1": 100, "critical": 100, "high": 100 }}\'
docker run -v $WORKSPACE:/root/app tshethp/dssc-vulnerability-report:v4 --smartcheck-host dssc.bryceindustries.net --smartcheck-user administrator --smartcheck-password Trend@123 --insecure-skip-tls-verify --min-severity low dssc.bryceindustries.net:5000/cluster-service:latest
mv $WORKSPACE/DSSCReport.xlsx $WORKSPACE/ClusterService-Vulnerability-report.xlsx
curl -F file=@$WORKSPACE/ClusterService-Vulnerability-report.xlsx -F "initial_comment=DSSC report for Cluster service image" -F channels=buildnotification -H "Authorization: Bearer xoxp-375765158032-376600608661-776771608885-6093db05ce4db6b26c68ca36bc42c4cf" https://slack.com/api/files.upload
'''
            sh '''docker run -v /var/run/docker.sock:/var/run/docker.sock deepsecurity/smartcheck-scan-action --image-name location-service:latest --smartcheck-host="dssc.bryceindustries.net" --smartcheck-user="administrator" --smartcheck-password="Trend@123" --insecure-skip-tls-verify --insecure-skip-registry-tls-verify --preregistry-scan --preregistry-user admin --preregistry-password Trend@123 --findings-threshold \'{"malware": 100, "vulnerabilities": { "defcon1": 100, "critical": 100, "high": 100 }, "contents": { "defcon1": 100, "critical": 100, "high": 100 }, "checklists": { "defcon1": 100, "critical": 100, "high": 100 }}\'
docker run -v $WORKSPACE:/root/app tshethp/dssc-vulnerability-report:v4 --smartcheck-host dssc.bryceindustries.net --smartcheck-user administrator --smartcheck-password Trend@123 --insecure-skip-tls-verify --min-severity low dssc.bryceindustries.net:5000/location-service:latest
mv $WORKSPACE/DSSCReport.xlsx $WORKSPACE/LocationService-Vulnerability-report.xlsx
curl -F file=@$WORKSPACE/LocationService-Vulnerability-report.xlsx -F channels=buildnotification -H "Authorization: Bearer xoxp-375765158032-376600608661-776771608885-6093db05ce4db6b26c68ca36bc42c4cf" https://slack.com/api/files.upload'''
            script {
              slackSend color: "warning", message: "REPORT: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} created Cluster service and scanned with security tool. To get more details go to http://jenkins.bryceindustries.net:8080/job/DockerbaseDSSC/job/master/${env.BUILD_NUMBER}/execution/node/3/ws/ClusterService-Vulnerability-report.xlsx"

              slackSend color: "warning", message: "REPORT: ${env.JOB_NAME} with buildnumber ${env.BUILD_NUMBER} created Location service and scanned with security tool. To get more details go to http://jenkins.bryceindustries.net:8080/job/DockerbaseDSSC/job/master/${env.BUILD_NUMBER}/execution/node/3/ws/LocationService-Vulnerability-report.xlsx"
            }

          }
        }
        stage('Integration Test') {
          steps {
            sh '''echo "Integration test"
sleep 17'''
          }
        }
        stage('Unit test') {
          steps {
            sh '''echo "Unit test"
sleep 24'''
          }
        }
      }
    }
    stage('Classify image tag') {
      steps {
        sh '''echo "docker tag to classify image"
sleep 5'''
      }
    }
    stage('Push image stage') {
      steps {
        sh '''echo "ECR registry push for image"
sleep 10'''
      }
    }
    stage('Deploy to Staging') {
      steps {
        sh '''echo "ECS Application deployment started"
sleep 10'''
      }
    }
    stage('Manual Test Success?') {
      steps {
        input 'Deployment Approval based on manual testing'
      }
    }
    stage('Classify image for production') {
      parallel {
        stage('Classify image for production') {
          steps {
            sh '''docker tag cluster-service:latest bryce.azurecr.io/bryce/cluster-service:latest
sleep 10'''
          }
        }
        stage('Destroy VMware Environment ') {
          steps {
            sh '''# /usr/local/bin/terraform destroy -target aws_instance.shellshock_host --auto-approve
/usr/local/bin/terraform destroy --auto-approve'''
          }
        }
      }
    }
    stage('Approve for production deploy') {
      steps {
        input 'Approved for production deploy'
      }
    }
    stage('Push Prod Image') {
      parallel {
        stage('Push Prod Image') {
          steps {
            sh '''docker login bryce.azurecr.io -u bryce -p +3BMjKEDQVvWuODOMM4SR2iZ1LWtOUMo
docker push bryce.azurecr.io/bryce/cluster-service:latest'''
          }
        }
        stage('Push image to DR ECR region') {
          steps {
            sh '''echo "Pushing image to AWS DR region"
sleep 10'''
          }
        }
      }
    }
    stage('Virtual Patch Prod') {
      steps {
        sh '''echo "Deep Security virtual patching of server using recommendation scan"
sleep 10'''
      }
    }
    stage('White list Apps') {
      steps {
        sh '''echo "Deep Security Application control whitelist application"
sleep 5'''
      }
    }
    stage('Deplo to Prod') {
      steps {
        sh '''echo "Deploy application to production"
sleep 7'''
      }
    }
    stage('Stop Whitelist App') {
      steps {
        sh '''echo "Deep security stop whitelisting of app using Application control"
sleep 3'''
      }
    }
  }
}