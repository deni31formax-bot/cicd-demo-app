pipeline {
    agent { label 'linux' }

    options {
        skipDefaultCheckout()
    }

    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/deni31formax-bot/cicd-demo-app.git', trim: true)
        string(name: 'GIT_BRANCH', defaultValue: 'master', trim: true)
        booleanParam(name: 'RUN_TESTS', defaultValue: true)
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                git url: params.REPO_URL, branch: params.GIT_BRANCH
                script {
                    env.CHECKED_OUT_SHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo "Branch: ${params.GIT_BRANCH}, commit: ${env.CHECKED_OUT_SHA}"
                }
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    for file in README.md requirements.txt; do
                        test -f "$file" || { echo "Missing file: $file" >&2; exit 1; }
                    done
                    for directory in src tests; do
                        test -d "$directory" || { echo "Missing directory: $directory" >&2; exit 1; }
                    done
                '''
            }
        }

        stage('Test') {
            when { expression { params.RUN_TESTS } }
            steps {
                sh 'PYTHONDONTWRITEBYTECODE=1 python3 -m unittest discover -s tests -v'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    printf 'BUILD_NUMBER=%s\\nGIT_COMMIT=%s\\n' "$BUILD_NUMBER" "$CHECKED_OUT_SHA" > build-info.txt
                    tar -czf app.tar.gz -- README.md requirements.txt src tests build-info.txt
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'app.tar.gz', fingerprint: true
            }
        }
    }

    post {
        success { echo 'Build succeeded.' }
        unsuccessful { echo "Build finished with status ${currentBuild.currentResult}." }
    }
}
