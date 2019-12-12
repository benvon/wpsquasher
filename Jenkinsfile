
// redoing as pipeline
pipeline {
  agent {
    label 'mksquashfs'
  }
  options {
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '4'))
    lock resource: 'wpsquasher'
  }
  parameters {
    string(name: 'SITE', description: 'wordpress hostname for site being squashed', defaultValue: 'blog.benvon.net')
    string(name: 'IMG_HOST', description: 'where we will scp the file, username@host.tld', defaultValue: 'jenkins@192.168.1.12')
  }
  stages {
   stage ('Start notification'){
     steps {
       slackSend (color: '#FFFF00', message: 'Starting wpsquash job')
     }
   }
   stage ('Cleanup') {
     steps {
       sh "rm -rf ./*"
     }
   }
   stage ('Fetch WP archive') {
     steps {
       downloadWP()
     }
   }
   stage ('Prep WP directory') {
     steps {
       prepWPDir()
     }
   }
   stage ('Make Squash') {
     steps {
       makeSquashFS()
     }        
   }
   stage ('Upload Squash') {
     steps {
       uploadSquash()
     }
   }
   stage ('End notification') {
     steps {
       slackSend (color: '#00FF00', message: 'wpsquaser success!')
     }
   }
  } //end stages
} // end pipeline

def downloadWP() {
  sh "curl -O  https://wordpress.org/latest.tar.gz"
} // end downloadWP

def prepWPDir() {
  sh "tar xfvz latest.tar.gz"
  sh "rm -rf wordpress/wp-content"
  sh "ln -s /data/app/${params.SITE}/wp-content wordpress/wp-content"
  sh "ln -s /data/config/${params.SITE}/wp-config.php wordpress/wp-config.php"
} //end prepWPDir

def makeSquashFS() {
  sh "mksquashfs wordpress/ wordpress-autobuild-${BUILD_NUMBER}.squash -force-uid 5001 -force-gid 5001"
} //end makeSquashFS

def uploadSquash() {
  sh "scp -o StrictHostKeyChecking=no wordpress-autobuild-${BUILD_NUMBER}.squash ${params.IMG_HOST}:/data/images/${params.SITE}/"
  sh "ssh -o StrictHostKeyChecking=no ${params.IMG_HOST} 'ln -sf /data/images/${params.SITE}/wordpress-autobuild-${BUILD_NUMBER}.squash /data/images/${params.SITE}/wp_squash.latest'"
} // end uploadSquash
