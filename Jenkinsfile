pipeline {
    agent any

    environment {
        JAVA_HOME = tool name: 'JDK17', type: 'jdk'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"

        APP_DIR = "/opt/bus_booking/bus_booking"
        TOMCAT_DIR = "/opt/tomcat10"
        TOMCAT_VERSION = "10.1.30"
        WAR_NAME = "bus-booking-app-1.0-SNAPSHOT.war"
        APP_CONTEXT = "bus-booking-app"
    }

    stages {

        stage('Checkout Code') {
            steps {
                sh '''
                echo "=== Checking out Correct Repo & Branch ==="

                sudo rm -rf /opt/bus_booking
                sudo mkdir -p /opt/bus_booking
                cd /opt/bus_booking

                git clone -b feature-1 https://github.com/vivek-co/bus_booking.git bus_booking

                echo "=== Code Pulled ==="
                '''
            }
        }

        stage('Verify POM Location') {
            steps {
                sh '''
                echo "=== Listing files in APP_DIR ==="
                ls -l ${APP_DIR}

                if [ ! -f "${APP_DIR}/pom.xml" ]; then
                    echo "❌ ERROR: pom.xml not found in ${APP_DIR}"
                    exit 1
                fi
                
                echo "✅ POM found"
                '''
            }
        }

        stage('Build WAR') {
            steps {
                sh '''
                echo "=== Running Maven Build ==="
                cd ${APP_DIR}
                mvn clean package -DskipTests
                '''
            }
        }

        stage('Install Tomcat (if missing)') {
            steps {
                sh '''
                if [ ! -d "${TOMCAT_DIR}" ]; then
                    echo "=== Installing Tomcat 10 ==="
                    cd /opt
                    sudo wget https://archive.apache.org/dist/tomcat/tomcat-10/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz
                    sudo tar -xzf apache-tomcat-${TOMCAT_VERSION}.tar.gz
                    sudo mv apache-tomcat-${TOMCAT_VERSION} tomcat10
                    sudo chmod +x /opt/tomcat10/bin/*.sh
                else
                    echo "Tomcat is already installed"
                fi
                '''
            }
        }

        stage('Deploy WAR to Tomcat') {
            steps {
                sh '''
                echo "=== Deploying WAR ==="

                if [ ! -d "${TOMCAT_DIR}/webapps" ]; then
                    echo "❌ ERROR: Tomcat webapps folder NOT found!"
                    exit 1
                fi

                echo "Stopping Tomcat..."
                sudo ${TOMCAT_DIR}/bin/shutdown.sh || true
                sleep 5

                echo "Removing old deployment..."
                sudo rm -rf ${TOMCAT_DIR}/webapps/${APP_CONTEXT}*
                
                echo "Copying new WAR..."
                sudo cp ${APP_DIR}/target/${WAR_NAME} ${TOMCAT_DIR}/webapps/${APP_CONTEXT}.war

                echo "Starting Tomcat..."
                sudo ${TOMCAT_DIR}/bin/startup.sh
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "=== Waiting for Tomcat to start ==="
                sleep 10

                STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/${APP_CONTEXT}/)

                if [ "$STATUS" = "200" ]; then
                    echo "✅ Deployment Successful!"
                else
                    echo "❌ Deployment Failed! HTTP Status = $STATUS"
                    exit 1
                fi
                '''
            }
        }
    }
}
