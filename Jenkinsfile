pipeline {
  agent {
    label 'android'
  }
  options {
    // Stop the build early in case of compile or test failures
    skipStagesAfterUnstable()
  }
  stages {
    stage('Test and Build') {
        steps {
        dir('android_src/') {
            //git credentialsId: 'github-ssh-key', url: 'git@github.com:0xE2/simple-timestamp-app.git'
            git credentialsId: 'github-ssh-key', url: 'git@github.com:0xE2/diva-android.git'
            withSonarQubeEnv('android') {
                sh 'sudo docker system prune -f'
                sh 'sudo docker run -v "$PWD":/home/gradle/App -w /home/gradle/App android-build:android-gradle gradle sonarqube \
  -Dsonar.projectKey=adroid_tmstmp \
  -Dsonar.host.url=$SONAR_HOST_URL \
  -Dsonar.login=$SONAR_AUTH_TOKEN'
            }
            sh '''
                pwd
                sudo docker run -v "$PWD":/home/gradle/App -w /home/gradle/App android-build:android-gradle gradle assembleDebug
                '''
        }
        }
    }
    stage('Source code & apk security check') {
        steps {
            dir('android_src/') {
                sh '''
                    qark --java app/
                    cp /home/ubuntu/.local/lib/python2.7/site-packages/qark/report/report.html java_rev.html
                    '''
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/home/ubuntu/.local/lib/python2.7/site-packages/qark/report/', reportFiles: 'report.html', reportName: 'Java code review', reportTitles: ''])
                sh '''
                    qark --apk ./app/build/outputs/apk/debug/app-debug.apk
                    cp /home/ubuntu/.local/lib/python2.7/site-packages/qark/report/report.html apk_rev.html
                    '''
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/home/ubuntu/.local/lib/python2.7/site-packages/qark/report/', reportFiles: 'report.html', reportName: 'APK review', reportTitles: ''])
            }
        }
    }
    stage('Save artifacts and publish') {
        steps{
            archiveArtifacts artifacts: "**/app-debug.apk", excludes: "**/*unaligned.apk", fingerprint: true, onlyIfSuccessful: true
            sshPublisher(publishers: [sshPublisherDesc(configName: 'Prod', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''pwd
ls''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: 'apk', remoteDirectorySDF: false, removePrefix: 'android_src/app/build/outputs/apk/debug', sourceFiles: '**/app-debug.apk.apk')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            sshPublisher(publishers: [sshPublisherDesc(configName: 'Prod', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''sudo killall -9 /usr/bin/python3
cd ~/bot
sudo /usr/bin/python3 bot.py >log 2>&1 &''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: 'reports', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'java_rev.html, apk_rev.html')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
        }
    }
  }
  post {
      always {
          telegramSend "Running build ${env.BUILD_ID} for ${env.JOB_NAME}"
      }
      success {
          telegramSend "Build ${env.BUILD_ID} finished"
      }
      unsuccessful {
          telegramSend "Build ${env.BUILD_ID} for commit ${env.GIT_COMMIT} failed"
      }
  }
}
