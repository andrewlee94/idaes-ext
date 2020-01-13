pipeline {
  agent { 
    docker { 
      image 'conda/miniconda3-centos7:latest'
    } 
  }
  stages {
    stage('root-setup') {
      steps {
        slackSend (message: "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        sh 'yum install -y gcc g++ git gcc-gfortran libboost-dev make'
        dir('idaes-dev') {
          git url: 'https://github.com/makaylas/idaes-dev.git',
          credentialsId: '6ca01274-150a-4dd4-96ec-f0d117b0ea95'
        }
      }
    }
    stage('idaes-dev 3.7-setup') {
      steps {
        sh '''
         cd idaes-dev
         conda create -n idaes3.7 python=3.7 pytest
         source activate idaes3.7
         pip install -r requirements-dev.txt --user jenkins
         export TEMP_LC_ALL=$LC_ALL
         export TEMP_LANG=$LANG
         export LC_ALL=en_US.utf-8
         export LANG=en_US.utf-8
         python setup.py develop
         idaes get-extensions
         export LC_ALL=$TEMP_LC_ALL
         export LANG=$TEMP_LANG
         source deactivate
         '''
      }
    }
    // stage('idaes-ext build') {
    //   steps {
    //     sh 'cd ..'
    //     sh 'ls'
    //     sh 'bash scripts/compile_solvers.sh'
    //     sh 'bash scripts/compile_libs.sh'
    //     sh 'ls'
    //     sh 'ls dist-lib'
    //     ls 'dist-solvers'
    //   }
    // }
    stage('idaes-dev 3.7-test') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          sh '''
           source activate idaes3.7
           ls
           pwd
           pylint -E --ignore-patterns="test_.*" idaes || true
           pytest -c pytest.ini idaes -m "not nocircleci"
           source deactivate
           '''
        }
      }   
    }
  }
  post {
    success {
      slackSend (color: '#00FF00', message: "SUCCESSFUL - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }

    failure {
      slackSend (color: '#FF0000', message: "FAILED - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }
  }
}
