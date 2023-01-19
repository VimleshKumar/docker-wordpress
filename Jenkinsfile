pipeline {
    agent any
    environment {
        REPO = 'vimlesh/wordpress'
    }
    stages {
        stage ('Checkout: ${BRANCH_NAME}') {
            steps {
                echo "Checkout: ${BRANCH_NAME}: ${GIT_COMMIT}"
                script {
                    if ("${BRANCH_NAME}" == "master"){
                        TAG   = "latest"
                        NGINX = "nginx"
                        FPM   = "fpm"
                        CLI   = "cli"
                    }
                    else {
                        TAG   = "${BRANCH_NAME}"
                        NGINX = "${BRANCH_NAME}-nginx"
                        FPM   = "${BRANCH_NAME}-php7.1-fpm"
                        CLI   = "${BRANCH_NAME}-cli"                      
                    }
                }
                sh 'printenv'
            }
        }
        stage ('Build: Docker Micro-Service') {
            parallel {
                stage ('Wodpress Nginx'){
                    agent { label 'docker'}
                    steps {
                        sh "docker build -f nginx/Dockerfile -t ${REPO}:${TAG}-nginx nginx/"
                    }
                }
                stage ('Wordpress PHP-FPM') {
                    agent { label 'docker'}
                    steps {
                        sh "docker build -f php7-fpm/Dockerfile -t ${REPO}:${TAG}-fpm php7-fpm/"
                    }
                }
                stage ('Wordpress CLI') {
                    agent { label 'docker'}
                    steps {
                        sh "docker build -f cli/Dockerfile -t ${REPO}:${TAG}-cli cli/"
                    }
                }
            }
        }
        stage ('Deploy: Stop and clean Micro-Services'){
            agent { label 'docker'}
            steps {
                echo 'Remove micro-services stack'
                sh "docker rm -fv nginx-${BRANCH_NAME}"
                sh "docker rm -fv fpm-${BRANCH_NAME}"
                sh "docker rm -fv memcached-${BRANCH_NAME}"
                sh "docker rm -fv mariadb-${BRANCH_NAME}"
                sleep 10
                sh "docker network rm wordpress-micro-${BRANCH_NAME}"
            }
        }
        stage ('Deploy: Start Micro-Services'){
            agent { label 'docker'}
            steps {
                // Create Network
                sh "docker network create wordpress-micro-${BRANCH_NAME}"
                // Start database
                sh "docker run -d --name 'mariadb-${BRANCH_NAME}' -e MYSQL_ROOT_PASSWORD=wordpress -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=wordpress -e MYSQL_DATABASE=wordpress --network wordpress-micro-${BRANCH_NAME} mariadb:latest"
                sleep 15
                // Start Memcached
                sh "docker run -d --name 'memcached-${BRANCH_NAME}' --network wordpress-micro-${BRANCH_NAME} memcached"
                // Start application micro-services
                sh "docker run -d --name 'fpm-${BRANCH_NAME}' --link mariadb-${BRANCH_NAME}:mariadb --link memcached-${BRANCH_NAME}:memcached --network wordpress-micro-${BRANCH_NAME} -v wordpress-micro-data:/var/www/html ${REPO}:${TAG}-fpm"
                sh "docker run -d --name 'nginx-${BRANCH_NAME}' --link fpm-${BRANCH_NAME}:wordpress --link memcached-${BRANCH_NAME}:memcached --network wordpress-micro-${BRANCH_NAME} -v wordpress-micro-data:/var/www/html ${REPO}:${TAG}-nginx"
            }
        }
        stage ('Test'){
            parallel {
                stage ('Micro-Services'){
                    agent { label 'docker'}
                    steps {
                        sleep 20
                        sh "docker logs nginx-${BRANCH_NAME}"
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Run regardless of the completion status of the Pipeline run.'
        }
        changed {
            echo 'Only run if the current Pipeline run has a different status from the previously completed Pipeline.'
        }
        success {
            echo 'Only run if the current Pipeline has a "success" status, typically denoted in the web UI with a blue or green indication.'
        }
        failure{
            echo 'Only run if the current Pipeline has a "failure" status, typically denoted in the web UI with a red indication.'
            echo 'Remove micro-services stack on failure'
            sh "docker rm -fv nginx-${BRANCH_NAME}"
            sh "docker rm -fv fpm-${BRANCH_NAME}"
            sh "docker rm -fv memcached-${BRANCH_NAME}"
            sh "docker rm -fv mariadb-${BRANCH_NAME}"
            sleep 10
            sh "docker network rm wordpress-micro-${BRANCH_NAME}"
        }
        unstable {
            echo 'Only run if the current Pipeline has an "unstable" status, usually caused by test failures, code violations, etc. Typically denoted in the web UI with a yellow indication.'
        }
        aborted {
            echo 'Only run if the current Pipeline has an "aborted" status, usually due to the Pipeline being manually aborted. Typically denoted in the web UI with a gray indication.'
        }
    }
}