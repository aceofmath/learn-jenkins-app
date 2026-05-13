pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
            reuseNode true
        }
    }

    environment {
        NETLIFY_SITE_ID = 'c89b4303-c3d2-4811-b789-d4f7bbb04d2b'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            
            steps {
                sh '''
                    echo '트리거 테스트 중...'
                    ls -al
                    node --version
                    npm --version
                    npm ci --force
                    npm run build
                    ls -al
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Test stage'
                sh '''
                    test -f build/index.html
                    npm test
                '''
            }
        }

        stage('E2E') {
            steps {
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build & sleep 10
                    npx playwright test --reporter=html
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "프로젝트 배포중... 사이트 아이디 : $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }

    }
}
