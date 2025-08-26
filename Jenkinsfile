pipeline {
  agent any

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '20'))
    disableConcurrentBuilds()
    timeout(time: 15, unit: 'MINUTES')
  }

  environment {
    AWS_REGION = 'us-east-1'
    S3_BUCKET  = 'jenkins-use'           
    SITE_URL   = 'http://jenkins-use.s3-website-us-east-1.amazonaws.com'     
  }

  stages {

    stage('Lint HTML') {
      steps {
        sh '''
          set -e
          echo "Running HTML linter (tidy)…"
          # Lint all html at repo root; fail on errors
          tidy -q -e *.html | tee tidy-report.txt
        '''
      }
    }

    stage('Upload to AWS') {
      steps {
        retry(3) {
          sh '''
            set -e
            echo "Uploading index.html to S3…"
            aws s3 cp index.html s3://$S3_BUCKET/index.html --acl public-read --cache-control no-cache
          '''
        }
      }
    }

    stage('Smoke Test (curl)') {
      steps {
        retry(5) {
          sh '''
            set -e
            echo "Waiting for propagation…"
            sleep 3
            echo "Hitting $SITE_URL"
            # 1) ensure HTTP 200
            curl -sS -I "$SITE_URL" | head -n1 | grep -q "200"
            # 2) ensure body contains expected text
            curl -sS "$SITE_URL" | grep -F "Static HTML Site"
          '''
        }
      }
    }
  }

  post {
    always { archiveArtifacts artifacts: 'tidy-report.txt', allowEmptyArchive: true }
    success { echo "✅ Deployed to $SITE_URL" }
    failure { echo "❌ Build failed. See console & tidy-report.txt" }
    cleanup { cleanWs() }
  }
}
