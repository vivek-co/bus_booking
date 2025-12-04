pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {
        JAVA_HOME = tool(name: 'JDK17', type: 'jdk')
        PATH = "${JAVA_HOME}/bin:${env.PATH}"

        BASE_DIR = "/opt/bus_booking"
        APP_DIR = "/opt/bus_booking/bus_booking"     // your repo folder
        TOMCAT_DIR = "/opt/tomcat10"
        WAR_NAME = "bus-booking-app.war"
    }

    stages {

        stage('Checkout Code') {
            steps {
                sh '''
                cd ${BASE_DIR}

                if [ -d "bus_booking/.git" ]; then
                    echo "=== Repo exists. Pulling latest ==="
                    cd bus_booking
                    git fetch --all
                    git reset --hard origin/main
                else
                    echo "=== Cloning repo ==="
                    git clone https://github.com/patilsahana1234/bus_booking.git
                fi
                '''
            }
        }

        stage('Build with Maven') {
            steps {
                sh '''
                echo "=== Checking POM location ==="
                ls -l ${APP_DIR}

                echo "=== Running Maven Build ==="
                cd ${APP_DIR}
                mvn clean package -DskipTests
                '''
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                echo "=== Stopping Tomcat ==="
                sudo ${TOMCAT_DIR}/bin/shutdown.sh || true
                sleep 5

                echo "=== Removing old WAR ==="
                sudo rm -f ${TOMCAT_DIR}/webapps/${WAR_NAME}
                sudo rm -rf ${TOMCAT_DIR}/webapps/${WAR_NAME%.war}

                echo "=== Copying new WAR ==="
                sudo cp ${APP_DIR}/target/*.war ${TOMCAT_DIR}/webapps/${WAR_NAME}

                echo "=== Starting Tomcat ==="
                sudo ${TOMCAT_DIR}/bin/startup.sh
                sleep 5
                '''
            }
        }

    }

    post {
        success {
            echo "Deployment successful! Application running at port 8081"
        }
        failure {
            echo "Build failed. Check logs."
        }
    }
}
