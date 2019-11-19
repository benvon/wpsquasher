node ('master') {
    try {
        slackSend (color: '#FFFF00', message: 'Starting wpsquash job')
        stage ('Workdir cleanup'){
          step([$class: 'WsCleanup'])
        }
        /*
        stage ('Code Checkout') {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']],
              userRemoteConfigs: [[url: 'git@github.com:benvon/wpsquasher.git',
                                 credentialsId: 'benvon_net_jenkins_ssh']]
          ])
        }
        */
        stage ('Fetch WP archive') {
          sh "curl -O https://wordpress.org/latest.tar.gz"
        }
        stage ('Unroll and prep WP dir') {
          sh "tar xfvz latest.tar.gz"
          sh "rm -rf wordpress/wp-content"
          sh "ln -s /data/app/${SITE}/wp-content wordpress/wp-content"
          sh "ln -s /data/config/${SITE}/wp-config.php wordpress/wp-config.php"
        }
      	stage ('Create squash') {
            sh "mksquashfs wordpress/ wordpress-autobuild-${BUILD_NUMBER}.squash -force-uid 5001 -force-gid 5001"
      	}
        stage ('Upload squash to repo') {
          sh "scp -o StrictHostKeyChecking=no wordpress-autobuild-${BUILD_NUMBER}.squash ${IMG_HOST}:/data/images/${SITE}/"
          sh "ssh -o StrictHostKeyChecking=no ${IMG_HOST} 'ln -sf /data/images/${SITE}/wordpress-autobuild-${BUILD_NUMBER}.squash /data/images/${SITE}/wp_squash.latest'"
        }
        slackSend (color: '#00FF00', message: 'wpsquaser success!')
        currentBuild.result = 'SUCCESS'
    } catch (err) {
        slackSend (color: '#FF0000', message: 'wpsquaser FAILED!!!')
        currentBuild.result = 'FAILED'
        throw err
    }
}
