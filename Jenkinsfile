pipeline{
    agent any
     triggers{
              pollSCM("H/5 * * * *")
         } //poll every 5 min
    stages {
        stage("Deliver to Docker Hub") {
            steps {
                 sh "docker build . -t snowboy420/frontend-calc"
                 withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerHub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']])
                 {
                     sh 'docker login -u ${USERNAME} -p ${PASSWORD}'
                 }
                 sh "docker push snowboy420/frontend-calc"
            }
        }
        stage("Selenium grid setup") {
            steps{
                sh "docker-compose -f selenium.yml up -d"
                //sh "docker network create SE3"
                //sh "docker run -d --rm -p 4444:4444 --net=SE --name selenium-hub selenium/hub"
                //sh "docker run -d --rm --net=SE3 -e HUB_HOST=selenium-hub --name selenium-node-firefox"
                //sh "docker run -d --rm --net=SE3 -e HUB_HOST=selenium-hub --name selenium-node-chrome"
                //sh "docker run -d --rm --net=SE3 --name frontend-cont-1 snowboy420/frontend-calc"
            }
        }
        stage("Execute system tests") {
            steps {
                sh "selenium-side-runner-server http://localhost:4444/wd/hub -c 'browserName=firefox' --base-url http://frontend-cont-1 test/system/CalcFunctionalTests.side"
                sh "selenium-side-runner-server http://localhost:4444/wd/hub -c 'browserName=chrome' --base-url http://frontend-cont-1 test/system/CalcFunctionalTests.side"
            }
        }
    }
   post {
       cleanup {
           echo "Cleaning the Docker environment"
           sh script: "docker-compose -f selenium.yml down", returnStatus:true
           //sh script:"docker stop selenium-hub", returnStatus:true
           //sh script:"docker stop selenium-node-firefox", returnStatus:true
           //sh script:"docker stop selenium-node-chrome", returnStatus:true
           //sh script:"docker stop app-test-container", returnStatus:true
           //sh script:"docker network remove SE3", returnStatus:true
       }
   }
}