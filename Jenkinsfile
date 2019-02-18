pipeline {

    agent any
    
    stages {

        stage('Build') {
            steps {
                echo 'Build'
                sh "mvn --batch-mode package" 
            }
        }

        stage('Archive Unit Tests Results') {
            steps {
                echo 'Archive Unit Test Results'
               step([$class: 'JUnitResultArchiver', testResults: 'target/surefire-reports/TEST-*.xml'])
            }
        }
        
        stage('Publish Unit Test results report') {
            steps {
                echo 'Report'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: false, reportDir: 'target/site/jacoco/', reportFiles: 'index.html', reportName: 'jacaco report', reportTitles: ''])

             }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploy'
                sh '''
                    for runName in `docker ps | grep "alpine-petclinic" | awk '{print $1}'`
                    do
                        if [ "$runName" != "" ]
                        then
                            docker stop $runName
                        fi
                    done
                    docker build -t alpine-petclinic -f Dockerfile.deploy .
                    docker run --name alpine-petclinic --rm -d -p 9966:8080 alpine-petclinic 
                '''
            }
        }
        stage("selenium test") {
       steps {
                echo 'Invoking automated test cases in docker ' 
                echo 'stop the docker container'
				sh "chmod +x robot.sh"
				sh "./robot.sh stop-test dev"
				echo 'build the docker selenium container'  
				sh "./robot.sh rebuild-test dev"
				echo 'start docker container' 
				sh "./robot.sh start-test dev"
				sh "set +e"
				sh "rm -rf report"
				sh "set -e"
				sh "mkdir report"
				echo 'copy test cases report from docker container'
				sh "docker cp selenium-testsuite:home/robotframework/src/test/selenium-robot/output.xml report"
				sh "docker cp selenium-testsuite:home/robotframework/src/test/selenium-robot/log.html report"
				sh "docker cp selenium-testsuite:home/robotframework/src/test/selenium-robot/report.html report"
            }
      }    
      
        stage('Archive Automation Test Results') {
            steps {
                echo 'Archive Automation Test Results'
               step([$class: 'RobotPublisher', disableArchiveOutput: false, logFileName: 'log.html', otherFiles: '', outputFileName: 'output.xml', outputPath: 'report/', passThreshold: 100, reportFileName: 'report.html', unstableThreshold: 0]);
            }
        }   
    }
}
