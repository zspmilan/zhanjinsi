#!/usr/bin/env groovy

pipeline {
  agent none
  options { 
    timestamps() 
    buildDiscarder(logRotator(numToKeepStr:'10'))
  }
  stages {
    stage('build'){
    agent { 
      node { 
        label 'master'
        customWorkspace '/tmp/jksdemo'
      }
    }
      steps {
        sh 'docker build -t docker.io/zspmilan/centos-jkmd:v4.0 .'
      }
    }
    stage('push') {
      agent { 
        node {
          label 'master'
        }
      }
      options { 
        retry (3) 
        skipDefaultCheckout()
      }
      steps {
        sh 'docker push docker.io/zspmilan/centos-jkmd:v4.0'
      }
    }
    stage('run') {
      agent { 
        node {
          label 'client'
          customWorkspace '/tmp/jksdemo'
        }
      }
      options { 
        retry (3) 
        skipDefaultCheckout()
      }
      steps {
        sh '''
           timestamp=$(date +%Y%m%d%H%M%S)
           docker run -d -p 8808:80 --name centos-jksmd_${timestamp} zspmilan/centos-jkmd:v4.0 /usr/sbin/init
           docker exec centos-jksmd_${timestamp} /srv/inint.sh
           echo ${timestamp} > timestamp
        '''
      }
    }
    stage('test') {
      agent { node { label 'client' } }
      options { skipDefaultCheckout() }
      steps {
        sh 'curl http://127.0.0.1:8808'
      }
    }
    stage('post_clear') {
      agent {
        node {
          label 'client'
          customWorkspace '/tmp/jksdemo'
        }
      }
      options { skipDefaultCheckout() }
      steps {
        sh '''
           timestamp=$(cat timestamp)
           docker stop centos-jksmd_${timestamp} && docker rm centos-jksmd_${timestamp}
        '''
      }
      post {
        always {
         echo 'Now will clear the workspace!'
         deleteDir()
        }
      }
    }
    stage('deploy') {
      agent {
        node {
          label 'CA'
          customWorkspace '/tmp/jksdemo'
        }
      }
      options { skipDefaultCheckout() } 
      steps {
        sh '''
           timestamp=$(date +%Y%m%d%H%M%S)
           docker run -d -p 8808:80 --name centos-jksmd_${timestamp} zspmilan/centos-jkmd:v4.0 /usr/sbin/init
           sleep 10s
           docker exec centos-jksmd_${timestamp} /srv/inint.sh
        ''' 
      } 
    }
    stage('application') {
      agent { node { label 'master' } } 
      options { skipDefaultCheckout() }
      steps {
        sh 'curl http://100.98.101.26:8808'
      }
    }
  }
  post {
    success {
      echo 'Everything is fine!'
      mail to: 'zspmilan@163.com 45478401@qq.com',
      subject: "Update the Pipeline result.",
      body: "The ${env.JOB_NAME} ${env.BUILD_NUMBER} is successful."
    }
   failure {
      echo 'Ops, it failed!'
      mail to: 'zspmilan@163.com',
      subject: "Update the Pipeline result.",
      body: "The ${env.JOB_NAME} ${env.BUILD_NUMBER} is failed!"
   }
  }
}
