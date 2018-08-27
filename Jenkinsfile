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
            TAG_CONSOLE="$(ws app get-tag console)"

            docker tag "$(ws app get-image web)"      "$TAG_WEB"
            docker tag "$(ws app get-image console)"  "$TAG_CONSOLE"

            docker push "$TAG_WEB"
            docker push "$TAG_CONSOLE"

            docker image rm "$TAG_WEB" "$TAG_CONSOLE"
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

            ws app docker-stack-config preview > docker-compose.yml

            export SLUG="$(echo $BRANCH_NAME | iconv -t ascii//TRANSLIT | sed -r \'s/[~\\^]+//g\' | sed -r \'s/[^a-zA-Z0-9]+/-/g\' | sed -r \'s/^-+\\|-+$//g\' | tr A-Z a-z)"
            export STACK="acme-${SLUG}"

            source $DOCKER_DAEMON_CONFIG
            source $DOCKER_STACK_CONFIG

            echo $REGISTRY_PASS | docker login quay.io/inviqa/acme --username $REGISTRY_USER --password-stdin
            docker stack deploy -c docker-compose.yml $STACK --with-registry-auth

          '''
        }
      }
    }
  }
  post { always { sh 'ws destroy' } }
}
