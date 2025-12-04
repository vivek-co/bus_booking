pipeline {
    agent { label 'vivek' }

        tools {
        maven 'maven'
        jdk 'JDK17'
    }

    
    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Run Application') {
            steps {
                sh '''
                    echo "Starting Spring Boot with nohup..."
                    nohup mvn spring-boot:run > app.log 2>&1 &
                    echo $! > app.pid
                    sleep 10
                '''
            }
        }

       stage('Validate Application') {
    steps {
        sh '''
            echo "Waiting for app on 8080..."

            i=1
            while [ $i -le 20 ]; do
                STATUS=$(curl --write-out "%{http_code}" --silent --output /dev/null http://localhost:8080)

                if [ "$STATUS" -eq 200 ]; then
                    echo "Application is UP! HTTP 200"
                    exit 0
                fi

                echo "Attempt $i/20: App not ready (HTTP $STATUS)... retrying"
                i=$((i + 1))
                sleep 3
            done

            echo "ERROR: Application FAILED to start!"
            exit 1
        '''
    }
}

        stage('Wait for 2 minutes') {
            steps {
                sleep(time: 2, unit: 'MINUTES')
            }
        }

        stage('Stop Application') {
            steps {
                sh '''
                    if [ -f app.pid ]; then
                        PID=$(cat app.pid)
                        echo "Stopping app with PID $PID"
                        kill $PID || true
                    fi
                '''
            }
        }
    }

    post {
        always {
            echo "Cleaning up..."
        }
    }
}
