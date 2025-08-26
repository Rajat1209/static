pipeline {
  agent any
  stages {

    stage('Lint HTML') {
      steps {
        sh '''
          set -e
          echo "Running HTML linter (tidy)..."
          tidy -q -e *.html
        '''
      }
    }
    
    stage('Upload To AWS') {
      steps {
        sh 'echo "Hello World"'
        sh '''
            echo "Multiline shell steps works too"
            ls -lah
        '''
      }
    }

  }
}
