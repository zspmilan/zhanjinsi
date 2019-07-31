#!/usr/bin/env groovy

pipeline {
  agent none
  stages {
    stage('build'){
    agent { 
      node { 
        label 'master'
        customWorkspace '/tmp/jksdemo'
      }
    }
      steps {
        sh 'docker build -t docker.io/zspmilan/centos-jkmd:v1.0 .'
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
        sh 'docker push docker.io/zspmilan/centos-jkmd:v1.0'
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
        sh 'docker run -d -p 8809:80 --name centos-jksmd zspmilan/centos-jkmd:v1.0 /usr/sbin/init'
        sh 'docker exec centos-jksmd systemctl start nginx'
      }
    }
    stage('test') {
      agent { node { label 'client' } }
      options { skipDefaultCheckout() }
      steps {
        sh 'curl http://127.0.0.1:8809'
      }
    }
  }
  post {
    success {
      echo 'Everything is fine!'
    }
  }
}
