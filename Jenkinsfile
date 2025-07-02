node { 
    docker.image('node:lts-buster-slim').withRun('-p 5000:5000') { builder ->
        
        env.CI = 'true'
  
        stage('Build') {
            echo 'Starting Build stage...'
            builder.inside { 
                sh 'npm install'
            }
            echo 'Build stage finished.'
        }

        stage('Test') {
            echo 'Starting Test stage...'
            builder.inside {
                sh './jenkins/scripts/test.sh'
            }
            echo 'Test stage finished.'
        }

        stage('Deliver') {
            echo 'Starting Deliver stage...'
            builder.inside {
                sh './jenkins/scripts/deliver.sh'
            }
       
            input message: 'Finished using the website? (Click "Proceed" to continue)'
            builder.inside {
                sh './jenkins/scripts/kill.sh'
            }
            echo 'Deliver stage finished.'
        }
    }
    echo 'Pipeline completed!'
}