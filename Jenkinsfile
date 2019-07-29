#!/usr/bin/env groovy

pipeline {
  agent none
  stages {
    stage('build'){
    checkout scm
    agent { 
      node { 
        label 'master'
        customWorkspace '/tmp/jksdemo'
      }
    }
      steps {
        sh 'docker build -t centos-jkmd:v1.0 .'
        sh 'docker push centos-jkmd:v1.0'
      }
    }
    stage('run') {
      agent { 
        node {
          label 'client'
          customWorkspace '/tmp/jksdemo'
        }
      }
      steps {
        sh 'docker run -d -p 8809:80 --name centos-jksmd centos-jkmd:v1.0 /usr/sbin/init'
      }
    }
    stage('test') {
      agent { node { label 'client' } }
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
