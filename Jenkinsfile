pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('gitlantis-dockerhub')
    }
    stages {
        stage('Push staging to dockerhub') {
            when {
                branch 'staging'               
            }
            steps {
                script {
                    SECRET_FILE_PATH = credentials([file(credentialsId: 'appsettings.json')])
                }
                sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker build -t gitlantis/user-test-api-dev:latest -f Dockerfile .
                    docker push gitlantis/user-test-api-dev:latest 
                    docker logout
                    docker run --rm -p 5000:5000 -p 80:8081 -e ASPNETCORE_HTTP_PORT=http://+:5000 gitlantis/user-test-api-dev:latest -v $SECRET_FILE_PATH:/App/appsettings.json
                '''
            }
        }
        stage('Push main to dockerhub') {
            when {
               branch 'main'               
            }
            steps {
                script {
                    SECRET_FILE_PATH = credentials([file(credentialsId: 'appsettings.json')])
                }
            print 'SECRET_FILE_PATH{ it }=' + SECRET_FILE_PATH.collect { it }

                sh '''
                    cat $DOCKERHUB_CREDENTIALS_PSW
                    cat $DOCKERHUB_CREDENTIALS_USR
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker build -t gitlantis/user-test-api-prod:latest -f Dockerfile . 
                    docker push gitlantis/user-test-api-prod:latest 
                    docker logout
                    docker run --rm -p 5000:5000 -p 80:80 -e ASPNETCORE_HTTP_PORT=http://+:5000 gitlantis/user-test-api-prod:latest -v $SECRET_FILE_PATH:/App/appsettings.json 
                '''
            }
        }        
    }
}