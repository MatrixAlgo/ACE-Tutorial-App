pipeline {
    agent any

    environment {
        ACE_NODE   = "ACE_NODE"
        ACE_SERVER = "IS01"
        BUILD_DIR  = "build"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build BAR') {
            steps {
                sh '''
                bash -lc '
                set -e
                set -x

                source /opt/IBM/ace-12.0.12.19/server/bin/mqsiprofile

                mkdir -p build

                BAR_NAME=$(basename $(pwd)).bar

                ibmint package \
                  --input-path $(pwd) \
                  --output-bar-file build/$BAR_NAME
                '
                '''
            }
        }

        stage('Deploy BAR') {
            steps {
                sh '''
                bash -lc '
                set -e
                set -x

                source /opt/IBM/ace-12.0.12.19/server/bin/mqsiprofile

                BAR_FILE=$(ls build/*.bar)

                mqsideploy ACE_NODE -e IS01 -a $BAR_FILE -w 120
                '
                '''
            }
        }
    }
}
