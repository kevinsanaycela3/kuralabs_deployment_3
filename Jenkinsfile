pipeline {
  agent any
   stages {
    stage('Build') {
      steps {
        sh '''#!/bin/bash
        python3 -m venv test3
        source test3/bin/activate
        pip install pip --upgrade
        pip install -r requirements.txt
        export FLASK_APP=application
        flask run &
        '''
      } 
    }
    stage('Test') {
      steps {
        sh '''#!/bin/bash
        python3 -m venv test3
        source test3/bin/activate
        py.test --verbose --junit-xml test-reports/results.xml
        '''
      }
      post{
        always {
          junit 'test-reports/results.xml'
        }
      } 
    }
  
   stage('Clean') {
      agent{label 'awsDeploy'}
      steps {
        sh '''#!/bin/bash
        if [[ $(ps aux | grep -i "gunicorn" | tr -s " " | head -n 1| cut -d " " -f 2) != 0 ]] 
        then
          ps aux | grep -i "gunicorn" | tr -s " " | head -n 1| cut -d " " -f 2 > pid.txt 
          kill $(cat pid.txt)
          exit 0
        fi
        ''' 
      }
    }
     
    stage('Deploy') {
      agent{label 'awsDeploy'}
      steps {
      keepRunning {
        sh '''#!/bin/bash
        pip install -r requirements.txt
        pip install gunicorn
        python3 -m gunicorn -w 4 application:app -b 0.0.0.0 --daemon
        '''
      }
     }
     post{
         success {
            slackSend(message: 
                      """
                      DEPLOYMENT SUCCESSFUL${custom_msg()}
                      """)
        }
         failure {
            slackSend(message: """
            DEPLOYMENT FAILED ${custom_msg()}
            """)
        }
       }
   
    }      
  }
}

def custom_msg()
{
  def JENKINS_LOG= 
    """
    Job: [${env.JOB_NAME}]
    Path to log of each step: http://3.89.115.40:8080/job/deployment3/indexing/console
    """
  return JENKINS_LOG
}

      
      
