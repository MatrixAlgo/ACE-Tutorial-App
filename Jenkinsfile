// Add Generic Webhook Trigger
properties([
    pipelineTriggers([
        [$class: 'GenericTrigger',
         genericVariables: [[key: 'ref', value: '$.ref']],
         token: 'ace-pipeline-123',                 // must match webhook URL token
         regexpFilterText: '$ref',
         regexpFilterExpression: 'refs/heads/main'  // trigger only on main branch
        ]
    ])
])

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

                # Name BAR uniquely per build to avoid conflicts
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

                # Pick the latest BAR file created
                BAR_FILE=$(ls -t build/*.bar | head -1)

                echo "Deploying $BAR_FILE to $ACE_NODE:$ACE_SERVER"

                mqsideploy $ACE_NODE -e $ACE_SERVER -a $BAR_FILE -w 120
                '
                '''
            }
        }
    }
}
