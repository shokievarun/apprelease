pipeline {
  agent {label 'master' }

  environment {
        RECIPIENTS = 'talktomanjunath248@gmail.com'
        START_SUBJECT = '${JOB_NAME} - Build # ${BUILD_NUMBER} Started'
        START_BODY = '''\
        Hi Team,<br><br>
        We have started the ${JOB_NAME}.<br><br>
        This is an auto-generated email.<br>
        Please send an email to the DevOps team in case of any concerns.<br><br>
        Thanks,<br>
        Jenkins DevOps
        '''
        SUCCESS_SUBJECT = '${JOB_NAME} - Build # ${BUILD_NUMBER} Succeeded'
        SUCCESS_BODY = '''\
        Hi Team,<br><br>
        The pipeline ${JOB_NAME} has completed successfully.<br><br>
        Build Number: ${BUILD_NUMBER}<br>
        Status: ${BUILD_STATUS}<br>
        APK drive path: <a href="https://drive.google.com/drive/folders/14a69gHHh7lvg_hfZaRL10KK6V3R6dArK?usp=sharing">APK Link</a><br><br>
        This is an auto-generated email.<br>
        Please send an email to the DevOps team in case of any concerns.<br><br>
        Thanks,<br>
        Jenkins DevOps
        '''
        FAILURE_SUBJECT = '${JOB_NAME} - Build # ${BUILD_NUMBER} Failed'
        FAILURE_BODY = '''\
        Hi Team,<br><br>
        The pipeline ${JOB_NAME} has failed.<br><br>
        Build Number: ${BUILD_NUMBER}<br>
        Status: ${BUILD_STATUS}<br>
        Build URL: <a href="${BUILD_URL}">Build Link</a><br><br>
        Check the build log for more details.<br><br>
        This is an auto-generated email.<br>
        Please send an email to the DevOps team in case of any concerns.<br><br>
        Thanks,<br>
        Jenkins DevOps
        '''
    }

  stages {
    
    stage('Build Started')
    {
      steps {

        emailext(
                to: "${env.RECIPIENTS}",
                subject: "${env.START_SUBJECT}",
                body: "${env.START_BODY}",
                mimeType: 'text/html'
            )

      }
    }
    
    stage('Checkout and build it') {

      agent {
    docker {
      image 'manju248/flutter_ci:latest'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }

      steps {
        
        git branch: 'devops', url: 'https://github.com/shokievarun/apprelease.git'
        
        sh '''
            sed -i -E 's/(version: ([0-9]+\\.[0-9]+\\.[0-9]+)\\+)([0-9]+)/echo "\\1$((\\3 + 1))"/e' pubspec.yaml
            cat pubspec.yaml
            flutter clean
            flutter pub get
            flutter build apk --release --verbose

            mv build/app/outputs/flutter-apk/app-release.apk build/app/outputs/flutter-apk/devops_prod_$(grep version: pubspec.yaml | awk '{print $2}').apk

            sed -i 's/prod/preprod/g' lib/values/api_end_points.dart
            flutter build apk --release --verbose
            mv build/app/outputs/flutter-apk/app-release.apk build/app/outputs/flutter-apk/devops_prepod_$(grep version: pubspec.yaml | awk '{print $2}').apk

            sed -i 's/preprod/sandbox/g' lib/values/api_end_points.dart
            flutter build apk --release --verbose
            mv build/app/outputs/flutter-apk/app-release.apk build/app/outputs/flutter-apk/devops_sandbox_$(grep version: pubspec.yaml | awk '{print $2}').apk

        '''
      }
    }

    stage('Upload it to Google Drive') {
      steps {
        sh 'echo passed'
        sh 'cp -f /var/lib/jenkins/workspace/FlutterAutomation/build/app/outputs/flutter-apk/* ~/output/'
        sh 'cp -f /var/lib/jenkins/workspace/FlutterAutomation@2/build/app/outputs/flutter-apk/* ~/output/'
        sh 'rm -rf ~/output/app*'
        sh 'rclone copy -P ~/output/ drive:/ '

        /*sh '''
          git config --global user.email "talktomanjunath248@gmail.com"
          git config --global user.name "ManjunathGanesanR1"
          git add pubspec.yaml
          git commit -m "Updated by JenkinsCI"
          git push origin devops
         ''' */
      }
    }
    
    
  }

  post {
        
        // Notification when the pipeline succeeds
        success {
            emailext(
                to: "${env.RECIPIENTS}",
                subject: "${env.SUCCESS_SUBJECT}",
                body: "${env.SUCCESS_BODY}",
                mimeType: 'text/html'
            )
        }

        // Notification when the pipeline fails, with build log attached
        failure {
            emailext(
                to: "${env.RECIPIENTS}",
                subject: "${env.FAILURE_SUBJECT}",
                body: "${env.FAILURE_BODY}",
                attachLog: true,  // Attach the build log
                mimeType: 'text/html'
            )
        }
    }
}
