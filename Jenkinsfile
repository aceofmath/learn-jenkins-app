pipeline {
    /* 
    4.4 Docker를 사용하여 jenkins에서 빌드
    "jenkins 관리 > Plugins > Available plugins " 에서 docker pipeline 설치
    */
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
        // 4.6 Jenkinsfile Build단계
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

        // 4.7 Jenkinsfile Test단계
        stage('Test') {
            steps {
                echo 'Test stage'
                sh '''
                    test -f build/index.html
                    npm test
                '''
            }
        }

        /*
        4.9 [실습] Playwright를 활용한 E2E 테스트
        agent > docker를 "image 'mcr.microsoft.com/playwright:v1.39.0-jammy'" 변경 (node 포함되어 있어서 변경해도 무방)
        
        ※ junit jest와 Playwright가 둘다 test-result라는 폴더를 사용함.
        
        (1) package.json 수정
        충돌을 막아주기 위하여 junit의 폴더명을 변경해야함. (package.json)
        jest-junit > outputDirectory(jest-results)

        (2) 4.8 junit 설정 변경
        test-results/junit.xml > jest-results/junit.xml
        
        */
        stage('E2E') {
            steps {
                /*
                4.10 [실습] Playwright 테스트 보고서 HTML로 발행하기
                npx playwright test --reporter=html

                * 보고서보기
                Workspace > playwright-report > intex.html

                ★ 보안상의 이유로 index.html이 보이지 않을 수 있을 경우 아래 설정을 jenkins에 추가
                위치 : Jenkins관리  > Script Console
                내용 : 
                */
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

        stage('Prod E2E') {
            environment {
                CI_ENVIRONMENT_URL = 'https://fabulous-bavarois-68b4de.netlify.app'
            }
            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
        }
    }

    /*
    4.8 [실습] Jenkins에서 JUnit 테스트 보고서 게시하기
    test-results/junit.xml
    jenkins dashboard에 노출
    jenkins 메뉴에 "Tests" 메뉴도 추가됨
    */
    post {
        always {
            junit 'jest-results/junit.xml'
        }

    }
}
