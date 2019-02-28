pipeline {
  agent {
    docker {
      image 'harmonyone/harmony-jenkins-agent:latest'
    }

  }
  options {
    withAWS(credentials:'harmony-s3')
  }
  stages {
    stage('MCL') {
      steps {
        sh '''
          curl -L -o mcl.zip 'https://s3-us-west-2.amazonaws.com/harmony-jenkins-artifacts/mcl/mcl-add_jenkins_pipeline:16@32b5022e759de1de3d07b431c0ac8ba708943c6d.zip'
          mkdir mcl
          unzip -p mcl.zip archive/build/mcl.cpio.xz | unxz | (cd mcl && exec cpio -idumv)
        '''
      }
    }
    stage('Configure') {
      steps {
        sh '''
          mcldir=$(pwd)/mcl
          mkdir -p build
          cd build
          CFLAGS="-I${mcldir}/include" CXXFLAGS="-I${mcldir}/include" LDFLAGS="-L${mcldir}/lib" cmake ..
        '''
      }
    }
    stage('Build') {
      steps {
        sh '''
          cd build
          make
        '''
      }
    }
    stage('Install') {
      steps {
        sh '''
          cd build
          rm -rf destdir
          make DESTDIR=`pwd`/destdir install
        '''
      }
    }
    stage('Package') {
      steps {
        sh '''
          cd build
          find destdir/usr/local -depth | \\
            sed -n \'s@^destdir/usr/local/@@p\' | \\
            tr \'\\n\' \'\\0\' | \\
            (cd destdir/usr/local && exec cpio -o0 -Hnewc -v) | \\
            xz -9 > bls.cpio.xz
        '''
      }
    }
    stage('Save Build') {
      steps {
        archiveArtifacts 'build/**'
      }
    }
    stage('Upload') {
      steps {
        script {
          s3Upload(
            file: 'build/bls.cpio.xz',
            bucket: 'harmony-jenkins-artifacts',
            path: 'bls.cpio.xz',
          )
        }
      }
    }
  }
}
