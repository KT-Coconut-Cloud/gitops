pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '20'))
  }

  environment {
    KUSTOMIZE   = 'registry.k8s.io/kustomize/kustomize:v5.7.1'
    KUBECONFORM = 'ghcr.io/yannh/kubeconform:v0.7.0'
    // CRD(Application, WorkflowTemplate 등) 스키마는 CRDs-catalog 사용
    CRD_SCHEMA  = 'https://raw.githubusercontent.com/datreeio/CRDs-catalog/main/{{.Group}}/{{.ResourceKind}}_{{.ResourceAPIVersion}}.json'
  }

  stages {
    stage('Prepare') {
      steps {
        script {
          env.SHA = sh(script: 'git rev-parse --short=7 HEAD', returnStdout: true).trim()
        }
      }
    }

    stage('Secret scan') {
      steps {
        sh '''
          docker run --rm -u "$(id -u):$(id -g)" -e HOME=/tmp -v "$PWD":/repo \
            zricethezav/gitleaks:v8.28.0 git /repo --redact --no-banner --exit-code 1
        '''
      }
    }

    stage('Validate manifests') {
      steps {
        // kustomize 빌드 결과와 단일 매니페스트를 스키마로 검사. 오타, 잘못된 필드는 여기서 실패
        sh '''#!/bin/bash
          set -euo pipefail
          validate() {
            docker run --rm -i -v "$PWD":/w -w /w "$KUBECONFORM" -strict -summary \
              -schema-location default -schema-location "$CRD_SCHEMA" "$@"
          }
          for d in apps/result-app pii-job; do
            echo "== $d"
            docker run --rm -v "$PWD":/w -w /w "$KUSTOMIZE" build "$d" | validate -
          done
          echo "== argocd, database"
          validate argocd database
        '''
      }
    }
  }

  post {
    success {
      script {
        if (env.BRANCH_NAME == 'main') { notifyDiscord('SUCCESS') }
      }
    }
    failure {
      script { notifyDiscord('FAILURE') }
    }
  }
}

def notifyDiscord(String status) {
  def msg = "${status} ${env.JOB_NAME} #${env.BUILD_NUMBER} (${env.SHA ?: '-'})\n${env.BUILD_URL}"
  writeFile file: 'discord.json', text: groovy.json.JsonOutput.toJson([content: msg])
  withCredentials([string(credentialsId: 'discord-webhook', variable: 'DISCORD_WEBHOOK')]) {
    sh 'curl -sS -m 10 -H "Content-Type: application/json" -d @discord.json "$DISCORD_WEBHOOK" || true; rm -f discord.json'
  }
}
