node {
    try {
        stage ('Code Checkout') {
          deleteDir()
          checkout([$class: 'GitSCM', branches: [[name: '*/master']],
              userRemoteConfigs: [[url: 'git@github.com:benvon/wpsquasher.git',
                                 credentialsId: 'benvon_net_jenkins_ssh']]
          ])
        }
        stage ('Fetch WP archive') {
          sh "curl -O https://wordpress.org/latest.tar.gz"
        }
        stage ('Unroll and prep WP dir') {
          sh "tar xfvz latest.tar.gz"
          sh "rm -rf wordpress/wp-content"
          sh "ln -s /data/app/benvon.net/wp-content wordpress/wp-content"
          sh "ln -s /data/config/benvon.net/wp-config.php wordpress/wp-config.php"
        }
      	stage ('Create squash') {
            sh "mksquashfs wordpress/ wordpress-autobuild-${BUILD_NUMBER}.squash -force-uid 5001 -force-gid 5001"
      	}
        stage ('Upload squash to repo') {
          sh "scp wordpress-autobuild-${BUILD_NUMBER}.squash 192.168.1.12:/data/images/"
          sh "ssh 192.168.1.12 'ln -sf /data/images/wordpress-autobuild-${BUILD_NUMBER}.squash /data/images/wp_squash.latest'"
        }
        currentBuild.result = 'SUCCESS'
    } catch (err) {
        currentBuild.result = 'FAILED'
        throw err
    }
}
