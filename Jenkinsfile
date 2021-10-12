//*** notes ***
// en cas d'erreur au build de type java.nio.file.AccessDeniedException
// sudo chown -R jenkins:jenkins /var/lib/jenkins/

pipeline {

  // https://www.jenkins.io/doc/book/pipeline/docker/
  // n.b: ajouter l'utilisateur jenkins au groupe docker => usermod -aG jenkins docker

  agent { 
    docker { 
      image 'node:14-alpine'
      args '-u root' // == docker run -u root (utile pour install surge globalement)
    } 
  }
  
  // https://www.jenkins.io/doc/pipeline/tour/environment/
  // https://www.jenkins.io/doc/book/using/using-credentials/#configuring-credentials
  environment {
    HOME = '.'
    STAGING_DOMAIN = 'chris-todobem-staging.surge.sh'
    PRODUCTION_DOMAIN = 'chris-todobem.surge.sh'
    SURGE_LOGIN = credentials('surge-login')
    SURGE_TOKEN = credentials('surge-token')
  }

  stages {

    stage('build') {
      steps {
        sh 'npm install'
        sh 'npm run build'
        sh 'test -f public/index.html'

        // éauivalent de production d'artefacts accessibles aux autres "stages"
        stash includes: 'public/*', name: 'app'
      }
    }

    stage('test') {
      steps {
        unstash 'app'
        sh 'grep "Bravo" public/index.html'
      }
    }

    stage('deploy staging') {
      when {
        branch 'master' // "only works on a multibranch Pipeline" (doc)
      }
      steps {
        unstash 'app'
        echo "Deploy to ${STAGING_DOMAIN}"
        sh 'npm install -g surge'
        sh 'surge --project public --domain $STAGING_DOMAIN'
      }
    }

    stage('deploy prod') {
      when {
        branch 'master'
      }
      input{
        message "Déployer en production ?"
      }
      steps {
        unstash 'app'
        echo "Deploy to ${PRODUCTION_DOMAIN}"
        sh 'npm install -g surge'
        sh 'surge --project public --domain $PRODUCTION_DOMAIN'
      }
    }

  }
}