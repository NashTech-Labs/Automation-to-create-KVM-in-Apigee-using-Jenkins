def envList = ['dev-env', 'qa-env', 'qa1-env', 'ts-env', 'ts2-env', 'ts3-env', 'perf1-env', 'perf2-env']
def orgList = ['APIGEE_ORGANISATION_NAME']

// Parameters Separated with Separator
properties([
    parameters([
        [
          $class: 'ChoiceParameter',
          choiceType: 'PT_SINGLE_SELECT',
          description: 'Select the Organisation',
          name: 'ORGANISATION',
          script: [
              $class: 'GroovyScript',
              script: [classpath: [], sandbox: true, script: "return ${orgList.inspect()}"]
          ]
        ],
        [
            $class: 'ChoiceParameter',
            choiceType: 'PT_CHECKBOX',
            description: 'Check the Environment Name from the List',
            name: 'ENVIRONMENT',
            script: [
                $class: 'GroovyScript',
                script: [ classpath: [], sandbox: true, script: "return ${envList.inspect()}"
                ]
            ]
        ],
         // Separator
        separator(name: "ADD_KVM", sectionHeader: "Add KVM",
          separatorStyle: "border-width: 0",
          sectionHeaderStyle: """
            background-color: #7ea6d3;
            text-align: left;
            padding: 4px;
            color: #343434;
            font-size: 22px;
            font-weight: normal;
            text-transform: uppercase;
            font-family: 'Orienta', sans-serif;
            letter-spacing: 1px;
            font-style: italic;
          """
        ),
        [
           $class: 'DynamicReferenceParameter', 
           choiceType: 'ET_FORMATTED_HTML', 
           description: 'Mention the KVM to create',
           name: 'KVM_FILE_NAME', 
           omitValueField: true,
           referencedParameters: 'ENVIRONMENT',
           script: [
               $class: 'GroovyScript', 
               fallbackScript: [
                   classpath: [],
                   sandbox: true,
                   script: 
                       'return [\'Error message\']'
               ], 
               script: [
                   classpath: [], 
                   sandbox: true,
                   script: 
                       """ 
                           html=""
                           if (ENVIRONMENT.contains('')){
                               html="<input name='value' value='' class='setting-input' type='text'>"
                           }
                           else {
                             
                               html="Enter value in KVM_FILE_NAME to enter the value"
                           }
                           return html
                       """
               ]
           ]
        ]
    ])
])

def selectedKvms = params.KVM_FILE_NAME.split(',')

pipeline {
    agent any
    environment {
          GCLOUD_DIR = "$JENKINS_HOME/google-cloud-sdk/bin"
          APIGEE_CLI_DIR = "$HOME/.apigeecli/bin"
    }
    stages {
        stage('Installing Dependencies') {
      steps {
          sh '''#!/bin/bash
                echo "Checking for pre-installed dependencies..."
                echo ""
                if [ ! -d "$GCLOUD_DIR" ]; then
                    echo "Installing GCloud CLI..."
                    echo ""
                    cd $JENKINS_HOME
                    curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-412.0.0-linux-x86_64.tar.gz
                    tar -xf google-cloud-cli-412.0.0-linux-*.tar.gz
                    ./google-cloud-sdk/install.sh -q
                    source $JENKINS_HOME/google-cloud-sdk/completion.bash.inc
                    source $JENKINS_HOME/google-cloud-sdk/path.bash.inc
                else--
                    echo "GCloud CLI is already Installed!"
                    echo ""
                fi

                if [ ! -d "$APIGEE_CLI_DIR" ]; then
                    echo "Installing Apigee CLI..."
                    echo ""
                    curl -L https://raw.githubusercontent.com/apigee/apigeecli/main/downloadLatest.sh | sh -
                    
                else
                    echo "Apigee CLI is already Installed!"
                    echo ""
                fi
             '''
      }
    }
        // Logging into GCloud
        stage('Logging into Google Cloud and Get Access Token') {
          steps {
            script {
                withCredentials([file(credentialsId: '<gcp_service_account>', variable: 'GOOGLE_SERVICE_ACCOUNT_KEY')]) {
                sh '${GCLOUD_DIR}/gcloud auth activate-service-account --key-file ${GOOGLE_SERVICE_ACCOUNT_KEY}'
                env.TOKEN = sh([script: "${GCLOUD_DIR}/gcloud auth print-access-token", returnStdout: true ]).trim()
              }
            }
          }
        }
         stage('Create KVM in Apigee') {
          steps {
           script {
          
          for (envs in selectedEnvs) {
              for (kvms in selectedKvms) {
                // List the kvm in selected env
                def kvm = "${kvms}"
                def command = "$APIGEE_CLI_DIR/apigeecli kvms list -o ${params.ORGANISATION} -e ${envs} -t ${env.TOKEN}"
                // Run the command and capture the output
                def output = sh(script: command, returnStdout: true).trim()

                // Check if the KVM exists in the output
                if (output.contains(kvm)) {

                  echo "KVM ${kvms} exists in ${envs} Apigee."
                  sh "$APIGEE_CLI_DIR/apigeecli kvms delete -o ${params.ORGANISATION} -e ${envs} -n ${kvms} -t ${env.TOKEN}"
                  sh "$APIGEE_CLI_DIR/apigeecli kvms create -o ${params.ORGANISATION} -e ${envs} -n ${kvms} -t ${env.TOKEN}"
                  sh "$APIGEE_CLI_DIR/apigeecli kvms entries import -o ${params.ORGANISATION} -e ${envs} -m ${kvms} -f ${WORKSPACE}/${kvm}.json -t ${env.TOKEN}"

                } else {
                  echo "KVM ${kvms} does not exist in ${envs} Apigee."
                  sh "$APIGEE_CLI_DIR/apigeecli kvms create -o ${params.ORGANISATION} -e ${envs} -n ${kvms} -t ${env.TOKEN}"
                  sh "$APIGEE_CLI_DIR/apigeecli kvms entries import -o ${params.ORGANISATION} -e ${envs} -m ${kvms} -f ${WORKSPACE}/${kvm}.json -t ${env.TOKEN}"
              }
            }
          }
        }
      }
          
    }
  }
}
  
