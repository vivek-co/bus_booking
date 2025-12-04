pipeline {
    agent { label 'vivek' }

    environment {
        MAVEN_HOME = "/usr/local/maven"
        PATH = "${MAVEN_HOME}/bin:${PATH}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh ''' 
                    echo "Building project using Maven..."
                    mvn clean install
                '''
            }
        }

        stage('Run Application') {
            steps {
                sh '''
                    echo "Starting Spring Boot application with nohup..."
                    nohup mvn spring-boot:run > app.log 2>&1 &
                    echo $! > app.pid
                    sleep 15   # allow app startup
                '''
            }
        }

        stage('Validate Application') {
            steps {
                sh '''
                    echo "Validating application on port 8080..."

                    # Try for ~60 seconds
                    for i in {1..20}; do
                        STATUS=$(curl --write-out "%{http_code}" --silent --output /dev/null http://localhost:8080)

                        if [ "$STATUS" -eq 200 ]; then
                            echo "Application is UP! HTTP 200"
                            exit 0
                        fi

                        echo "Attempt $i/20: App not ready (HTTP $STATUS)... retrying"
                        sleep 3
                    done

                    echo "ERROR: Application FAILED to start!"
                    echo "------ App Log ------"
                    tail -n 200 app.log || true
                    exit 1
                '''
            }
        }

        stage('Wait for 2 minutes') {
            steps {
                echo "App is running — waiting for 2 minutes..."
                sleep(time: 2, unit: 'MINUTES')
            }
        }

        stage('Stop Application') {
            steps {
                sh '''
                    if [ -f app.pid ]; then
                        PID=$(cat app.pid)
                        echo "Stopping application (PID: $PID)..."
                        kill $PID || true
                        sleep 5
                        echo "Stopped"
                    else
                        echo "app.pid not found, nothing to stop"
                    fi
                '''
            }
        }
    }

    post {
        always {
            echo "Cleaning up workspace..."
            sh '''
                rm -f app.pid || true
                rm -f app.log || true
            '''
        }
    }
}
