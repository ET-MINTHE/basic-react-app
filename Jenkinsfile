pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  tools {
    nodejs 'node-latest'
  }
  parameters {
    string(name: 'IMAGE_REPO_NAME', defaultValue: 'elhadjtahirouminthe/basic-react', description: '')
    string(name: 'LATEST_BUILD_TAG', defaultValue: 'latest', description: '')
    string(name: 'DOCKER_COMPOSE_FILENAME', defaultValue: 'docker-compose.yml', description: '')
    string(name: 'DOCKER_STACK_NAME', defaultValue: 'react_stack', description: '')
    booleanParam(name: 'NPM_RUN_TEST', defaultValue: true, description: '')
    booleanParam(name: 'PUSH_DOCKER_IMAGES', defaultValue: true, description: '')
    booleanParam(name: 'DOCKER_STACK_RM', defaultValue: false, description: 'Remove previous stack.  This is required if you have updated any secrets or configs as these cannot be updated. ')
  }
  stages {
    stage('npm install'){
      steps{
         bat "npm install"
      }
    }
    stage('npm test'){
	    when{
		    expression{
			    return params.NPM_RUN_TEST
		    }
	    }
	steps{
	  bat "npm test -- --coverage"	
	}
    }
    stage('npm build'){
      steps{
        bat "npm run build"
      }
    }
    stage('docker build'){
      environment {
        COMMIT_TAG = bat(returnStdout: true, script: 'git rev-parse HEAD').trim().take(7)
        BUILD_IMAGE_REPO_TAG = "${params.IMAGE_REPO_NAME}:${params.LATEST_BUILD_TAG}"
      }
      steps{
        bat "docker build . -t $BUILD_IMAGE_REPO_TAG"
       
      }
    }
    stage('docker push'){
      when{
        expression {
          return params.PUSH_DOCKER_IMAGES
        }
      }
      environment {
        COMMIT_TAG = bat(returnStdout: true, script: 'git rev-parse HEAD').trim().take(7)
        BUILD_IMAGE_REPO_TAG = "elhadjtahirouminthe/basic-react:latest"
      }
      steps{
        bat "docker push $BUILD_IMAGE_REPO_TAG"
        
      }
    }
    stage('Remove Previous Stack'){
      when{
        expression {
	  return params.DOCKER_STACK_RM
        }
      }
      steps{
        bat "docker stack rm ${params.DOCKER_STACK_NAME}"
	      
		      
      }
    }
    stage('Docker Stack Deploy'){
      steps{
        bat "docker stack deploy -c ${params.DOCKER_COMPOSE_FILENAME} ${params.DOCKER_STACK_NAME}"
      }
    }
  }
  post {
    always {
      bat 'echo "This will always run"'
    }
  }
}
