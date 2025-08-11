pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'c64db2bf-dd03-4919-a20f-1606d5cb8f46'
        NETLIFY_SITE_NAME = 'dazzling-douhua-b7f0b1'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        // 'nfp_5AhSc42F1JwSHiUTxXpYT8s4Qp2fAMmrf3be'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Run Tests') {
            parallel {
                stage('Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build & 
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    export NODE_TLS_REJECT_UNAUTHORIZED=0
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify link --name $NETLIFY_SITE_NAME
                    node_modules/.bin/netlify status --verbose 
                    ls -l 
                    echo "debug" 
                    ls -l /var/jenkins_home/workspace/learn-jenkins-app
                    node_modules/.bin/netlify deploy --prod --dir=build --auth=$NETLIFY_AUTH_TOKEN --site $NETLIFY_SITE_ID
                '''
            }
        }
    }
}
