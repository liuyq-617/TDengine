import hudson.model.Result
import jenkins.model.CauseOfInterruption
properties([pipelineTriggers([githubPush()])])
node {
    git url: 'https://github.com/liuyq-617/TDengine.git'
}

def buildNumber = env.BUILD_NUMBER as int
if (buildNumber > 1) milestone(buildNumber - 1)
milestone(buildNumber)

// def abortPreviousBuilds() {
//   def currentJobName = env.JOB_NAME
//   def currentBuildNumber = env.BUILD_NUMBER.toInteger()
//   def jobs = Jenkins.instance.getItemByFullName(currentJobName)
//   def builds = jobs.getBuilds()

//   for (build in builds) {
//     if (!build.isBuilding()) {
//       continue;
//     }

//     if (currentBuildNumber == build.getNumber().toInteger()) {
//       continue;
//     }

//     build.doKill()    //doTerm(),doKill(),doTerm()
//   }
// }
// //停止之前相同的分支。。
// abortPreviousBuilds()
def abort_previous(){
  def buildNumber = env.BUILD_NUMBER as int
  if (buildNumber > 1) milestone(buildNumber - 1)
  milestone(buildNumber)
}
def pre_test(){
    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                sh '''
                sudo rmtaos
                '''
    }
    sh '''
    
    cd ${WKC}
    rm -rf *
    cd ${WK}
    git reset --hard
    git checkout develop
    git pull
    cd ${WKC}
    rm -rf *
    mv ${WORKSPACE}/* .
    cd ${WK}
    export TZ=Asia/Harbin
    date
    rm -rf ${WK}/debug
    mkdir debug
    cd debug
    cmake .. > /dev/null
    make > /dev/null
    make install > /dev/null
    cd ${WKC}/tests
    '''
    return 1
}
pipeline {
  agent none
  
  environment{
      WK = '/var/lib/jenkins/workspace/TDinternal'
      WKC= '/var/lib/jenkins/workspace/TDinternal/community'
  }
  
  stages {
      
    
      stage('Parallel test stage') {
        when {
              changeRequest()
          }
      parallel {
        stage('python') {
          //only pr triggering the build.
          
          agent{label 'pytest'}
          steps {
            
            pre_test()
            sh '''
            cd ${WKC}/tests
            ./test-all.sh pytest
            date'''
          }
        }
        stage('test_b1') {
          // when {
          //     changeRequest()
          // }
          agent{label 'b1'}
          steps {            
            pre_test()
            sh '''
            cd ${WKC}/tests
            ./test-all.sh b1
            date'''
          }
        }

        stage('test_crash_gen') {
          // when {
          //     changeRequest()
          // }
          agent{label "b2"}
          steps {
            pre_test()
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                sh '''
                cd ${WKC}/tests/pytest
                ./crash_gen.sh -a -p -t 4 -s 2000
                '''
            }
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                sh '''
                cd ${WKC}/tests/pytest
                ./handle_crash_gen_val_log.sh
                '''
            }
            sh '''
            date
            cd ${WKC}/tests
            ./test-all.sh b2
            date
            '''
          }
        }

        stage('test_valgrind') {
          // when {
          //     changeRequest()
          // }
          agent{label "b3"}

          steps {
            pre_test()
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                sh '''
                cd ${WKC}/tests/pytest
                ./valgrind-test.sh 2>&1 > mem-error-out.log
                ./handle_val_log.sh
                '''
            }           
            sh '''
            date
            cd ${WKC}/tests
            ./test-all.sh b3
            date'''
          }
        }
      //  stage('python p2'){
      //   //  when {
      //   //       changeRequest()
      //   //   }
      //    agent{label "p2"}
      //    steps{
      //       print('ss')       
      //       sh '''
      //       date
      //       cd ${WKC}/tests
      //       ./test-all.sh p2
      //       date
      //       '''
      //    }
      //  } 
        
    }
  }
  }
   
}
