pipeline {
  agent any

  parameters {
    choice(
      name: 'nrEventDeploymentType',
      choices: ['BASIC', 'BLUE_GREEN', 'CANARY', 'OTHER', 'ROLLING', 'SHADOW'],
      description: 'Select the type of deployment that will be triggered by this build.'
    )
  }

  environment {
    // ID of the New Relic entity to which this build will be associated:
    nrEntityGuid = 'NDExNzIyMHxBUE18QVBQTElDQVRJT058MTA1OTY1MjkzNw'

    // ID of the Jenkins credentials-set that contains the New Relic API key:
    nrApiCredentialsGuid = 'fc15c043-cb25-4298-b579-265ba16b428e'

    // Set to either `true` or `false` depending on whether your data is being stored in the European region of New Relic:
    nrIsEuropean = false
  }

  stages {
    stage('Derive dynamic environment variables') {
      steps {
        script {
          // A URL to the changelog or, if not linkable, a list of changes:
          env.nrEventChangeLog = env.RUN_CHANGES_DISPLAY_URL

          // The commit identifier, for example, a Git commit SHA:
          env.nrEventCommitHash = sh (
            script: '''
            SHORTENED_COMMIT_HASH=$(echo "${GIT_COMMIT}" | head -c8)

            printf "${SHORTENED_COMMIT_HASH}"
            ''',
            returnStdout: true
          )

          // A link to the system that generated the deployment:
          env.nrEventDeeplink = env.RUN_DISPLAY_URL

          // A description of the deployment:
          env.nrEventDescription = sh (
            script: '''
            printf "Build ${BUILD_DISPLAY_NAME} of the '${JOB_NAME}' job completed successfully using Jenkins v${JENKINS_VERSION}."
            ''',
            returnStdout: true
          )

          // An identifier used to correlate two or more events:
          env.nrEventGroupId = env.JOB_BASE_NAME

          // The username of the deployer or bot:
          env.nrEventUser = currentBuild.getBuildCauses()[0].userId

          // The version of the deployed software, for example, something like v1.1.
          env.nrEventVersion = env.BUILD_ID
        }
      }
    }
    stage('Print out the variables available to the execution environment') {
      steps {
        sh "printenv"
      }
    }
    stage('Check for required parameters') {
      steps {
        sh '''
        if [ -z "${nrEntityGuid}" ]; then
            echo 'The "nrEntityGuid" environment-variable is required!'
            exit 1
        fi

        if [ -z "${nrApiCredentialsGuid}" ]; then
            echo 'The "nrApiCredentialsGuid" environment-variable is required!'
            exit 1
        fi

        if [ -z "${nrEventVersion}" ]; then
            echo 'The "nrEventVersion" environment-variable is required!'
            exit 1
        fi
        '''
      }
    }
    stage('Build the project') {
      steps {
        sh '''
        echo 'TODO: Add script to build the project!'
        '''
      }
    }
    stage('Record a Change-Tracking event to New Relic') {
      steps {
        script {
          step([$class: 'NewRelicDeploymentNotifier',
            notifications: [
              [
                apiKey: "${nrApiCredentialsGuid}",
                applicationId: '',
                european: "${nrIsEuropean}",
                entityGuid: "${nrEntityGuid}",
                changelog: "${nrEventChangeLog}",
                commit: "${nrEventCommitHash}",
                deeplink: "${nrEventDeeplink}",
                deploymentType: "${params.nrEventDeploymentType}",
                description: "${nrEventDescription}",
                groupId: "${nrEventGroupId}",
                user: "${nrEventUser}",
                version: "${nrEventVersion}",
                timestamp: ''
              ]
            ]
          ])
        }
      }
    }
  }
}
