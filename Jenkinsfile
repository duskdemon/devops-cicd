pipeline {
  agent {
      label 'docker'
  }
  stages {
    stage('First stage'){
      steps {
        git branch: 'main', credentialsId: '93c85424-0736-45fb-922a-f08755a48cfc', url: 'git@github.com:duskdemon/kibana-role.git'
        echo "I'm runing, chekedOut repo!"  
      }
    }
    stage('Second stage'){
      steps {
        echo "And I'm too"
      }
    }
  }
}
