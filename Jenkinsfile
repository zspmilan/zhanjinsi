#!/usr/bin/env groovy
    agent {
        docker {
            image 'centos'
            args '--name jenmd'
        }
    }
    stages {
        stage('test') {
            steps {
            sh """
               echo "I in the container now!"  
               hostname
            """
            }
        }
    }
}
