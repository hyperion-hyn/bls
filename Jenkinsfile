pipeline {
  agent {
    docker {
      image 'harmonyone/harmony-jenkins-agent:latest'
    }

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
    stage('Configure & Build') {
      steps {
        sh '''
          mkdir -p build
          cd build
          mcldir=$(pwd)/mcl
          CFLAGS="-I${mcldir}/include"
          CXXFLAGS="-I${mcldir}/include"
          LDFLAGS="-L${mcldir}/lib"
          export CFLAGS CXXFLAGS LDFLAGS
          cmake ..
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
  }
}
