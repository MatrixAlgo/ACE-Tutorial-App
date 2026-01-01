pipeline {
    agent any

    // Trigger the pipeline automatically when a push happens to main
    triggers {
        GenericTrigger(
            genericVariables: [[key: 'ref', value: '$.ref']],
            token: 'ace-pipeline-123',   // <-- your secret token
            regexpFilterText: '$ref',
            regexpFilterExpression: 'refs/heads/main' // trigger only for main branch
        )
    }

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

                # Unique BAR name per build
                BAR_NAME=$(basename $(pwd))_${BUILD_NUMBER}.bar

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

    post {
        always {
            echo "Cleaning workspace"
            cleanWs()
        }
    }
}
