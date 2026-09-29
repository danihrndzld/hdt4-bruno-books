// Declarative pipeline: start the books API, run the Bruno smoke suite
// against it, publish the JUnit report. `bru run` exits 0 on pass and 1 on
// an assertion failure; that exit code is the gate.
//
pipeline {
  agent any
  options {
    skipDefaultCheckout()
    timeout(time: 10, unit: 'MINUTES')
  }
  environment {
    API        = 'api.py'
    COLLECTION = 'bruno/smoke'
    BRU_ENV    = 'local'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Start API') {
      steps {
        // The API outlives this sh step; the post block kills it by pid.
        sh '''
          nohup uv run "$API" > api.log 2>&1 &
          echo $! > api.pid
          for i in $(seq 1 60); do
            curl -sf http://127.0.0.1:8000/health && exit 0
            sleep 1
          done
          cat api.log; exit 1
        '''
      }
    }
    stage('Smoke') {
      steps {
        // bru writes the report relative to the working directory and does
        // not create missing folders (exit 2), so the report stays in place.
        // --sandbox=developer: bru 4.2's default QuickJS sandbox fails at
        // random under CPU load ("reading 'newContext'"), a red build with
        // every assertion correct. Measured 9/30 runs vs 0/30.
        dir(env.COLLECTION) {
          sh 'bru run --env "$BRU_ENV" --sandbox=developer --reporter-junit results.xml'
        }
      }
    }
  }
  post {
    always {
      junit allowEmptyResults: true, testResults: "${env.COLLECTION}/results.xml"
      sh 'kill "$(cat api.pid)" 2>/dev/null || true'
    }
  }
}
