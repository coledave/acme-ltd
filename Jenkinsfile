pipeline {
  agent any
  environment {
    MY127WS_KEY = credentials('acme-my127ws-key1')
    MY127WS_ENV = "jenkins"
  }
  stages {
    stage('build') { steps { sh 'ws install' } }
    stage('test') {
      parallel {
        stage('quality')    { steps { sh 'ws exec composer test-quality'    }}
        stage('unit')       { steps { sh 'ws exec composer test-unit'       }}
        stage('acceptance') { steps { sh 'ws exec composer test-acceptance' }}
      }
    }
    stage('publish') {
      steps {
        withCredentials([
          usernamePassword(credentialsId: 'acme-registry-credentials', usernameVariable: 'REGISTRY_USER', passwordVariable: 'REGISTRY_PASS')
        ]) {
          sh '''#!/bin/bash
            echo $REGISTRY_PASS | docker login quay.io/inviqa/acme --username $REGISTRY_USER --password-stdin
            
            TAG_NGINX="$(ws app get-tag nginx)"
            TAG_PHP="$(ws app get-tag php-fpm)"
            TAG_CONSOLE="$(ws app get-tag console)"

            docker tag "$(ws app get-image nginx)"    "$TAG_NGINX"
            docker tag "$(ws app get-image php-fpm)"  "$TAG_PHP"
            docker tag "$(ws app get-image console)"  "$TAG_CONSOLE"

            docker push "$TAG_NGINX"
            docker push "$TAG_PHP"
            docker push "$TAG_CONSOLE"

            docker image rm "$TAG_NGINX" "$TAG_CONSOLE" "$TAG_PHP"
          '''
        }
      }
    }
    stage('deploy-preview') {
      steps {
        withCredentials([
          file(credentialsId: 'docker-preview-swarm', variable: 'DOCKER_DAEMON_CONFIG'),
          file(credentialsId: 'acme-preview-config', variable: 'DOCKER_STACK_CONFIG'),
          usernamePassword(credentialsId: 'acme-registry-credentials', usernameVariable: 'REGISTRY_USER', passwordVariable: 'REGISTRY_PASS')
        ]) {
          sh '''#!/bin/bash

            
          '''
        }
      }
    }
  }
  post { always { sh 'ws destroy' } }
}
