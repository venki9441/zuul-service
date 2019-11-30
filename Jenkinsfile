pipeline {
  agent any
  tools { 
        maven 'Maven'
  }
  environment {
        EUREKA_IPADDRESS = ""
    }
  stages {
    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace... */
      steps {
        checkout scm
      }
    }
    stage('Build') {
      steps {
        sh 'mvn -B -DskipTests clean package'
        sh 'echo $USER'
        sh 'echo whoami'
      }
    }
    stage('CreateInstance') {
      steps {
        ansiblePlaybook credentialsId: 'ab13a9b0-7986-420a-af3f-3048a2288ffd', installation: 'Anisble', inventory: '/home/ec2-user/hosts', playbook: '$WORKSPACE/createInstance.yaml'
      }
   }
   stage('DeployArtifact') {
      steps {
        ansiblePlaybook become: true, credentialsId: 'ab13a9b0-7986-420a-af3f-3048a2288ffd', installation: 'Anisble', inventory: '/tmp/hosts_eureka', extras: '-e WORKSPACE=$WORKSPACE -e host_key_checking=no', playbook: '$WORKSPACE/deployArtifact.yaml'
      }
   }
   stage('GetEurekaIP') {
      steps {
        script {
            def ip = readFile '/tmp/hosts_eureka'
            def FILENAME = ip.split(' ')[0]
            echo "IP Address is "+ FILENAME
            EUREKA_IPADDRESS = FILENAME
         }
      }  
   }
    stage('BuildDownStream') {
        parallel {
//          stage('Customer') {
//            steps
//            {
//              build job: 'OMS_CUSTOMER', parameters: [[$class: 'StringParameterValue', name: 'EUREKA_IPADDRESS', value: "${EUREKA_IPADDRESS}" ]]
//            }
//          }
          stage('ZUUL') {
            steps
            {
              build job: 'OMS_ZULU', parameters: [[$class: 'StringParameterValue', name: 'EUREKA_IPADDRESS', value: "${EUREKA_IPADDRESS}" ]]
            }
          }
          stage('Product') {
            steps
            {
              build job: 'OMS_PRODUCT', parameters: [[$class: 'StringParameterValue', name: 'EUREKA_IPADDRESS', value: "${EUREKA_IPADDRESS}" ]]
            }
          }

/*          stage('Order') {
#            steps
#            {
#              build job: 'OMS_ORDER', parameters: [[$class: 'StringParameterValue', name: 'EUREKA_IPADDRESS', value: "${EUREKA_IPADDRESS}" ]]
#            }
          }*/
        }
    }
  }
}
