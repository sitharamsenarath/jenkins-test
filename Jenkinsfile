pipeline {
  agent any
  environment {
    IMAGE = "ghcr.io/sitharamsenarath/jenkins-test"
    GHCR_PAT = credentials('DOCKER_IMAGE')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Detect Language & Generate Dockerfile') {
      steps {
        sh '''
        # Default to unknown
        echo "unknown" > language.txt

        # Detect Python
        if ls *.py 1> /dev/null 2>&1 || [ -f requirements.txt ]; then
          echo "python" > language.txt
        elif ls *.R 1> /dev/null 2>&1 || [ -f DESCRIPTION ]; then
          echo "R" > language.txt
        fi
        '''

        script {
          def lang = readFile('language.txt').trim()

          if (lang == 'python') {
            writeFile file: 'Dockerfile', text: '''
FROM python:3.9-slim
WORKDIR /app
COPY . /app
RUN pip install --no-cache-dir -r requirements.txt || true
CMD ["python", "app.py"]
'''
          } else if (lang == 'R') {
            writeFile file: 'Dockerfile', text: '''
FROM r-base:latest
WORKDIR /app
COPY . /app
RUN R -e "if(!requireNamespace(\\"renv\\", quietly=TRUE)) install.packages(\\"renv\\"); renv::restore()" || true
CMD ["Rscript", "app.R"]
'''
          } else {
            error("Unsupported or unknown language — no Dockerfile generated")
          }
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE .'
      }
    }

    stage('Push to GHCR') {
      steps {
        // Debug step: Check if GHCR_PAT is injected correctly
        sh 'echo "GHCR_PAT is: ${GHCR_PAT:+"[SET]"}"'
        sh 'echo "GHCR_PAT begins with: ${GHCR_PAT:0:4}"'

        // Login and push
        sh 'echo $GHCR_PAT | docker login ghcr.io -u sitharamsenarath --password-stdin'
        sh 'docker push $IMAGE'
      }
    }
  }
}
