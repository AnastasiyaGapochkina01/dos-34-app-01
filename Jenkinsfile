def remote = [:]
pipeline {
	agent {
	  label 'docker'
	}

	parameters {
	  booleanParam(name: 'RUN_TESTS', defaultValue: false)
	  gitParameter(name: 'BRANCH', type: 'PT_BRANCH', branchFilter: 'origin/(.*)', selectedValue: 'DEFAULT',)
	}

	environment {
	  REGISTRY = 'anestesia01/dos-34'
	  DOCKER_TOKEN = credentials('docker-token')
	  GIT_URL = 'git@github.com:AnastasiyaGapochkina01/dos-34-app-01.git'
	  HOST = '172.31.43.189'
	  PRJ_NAME = 'diary'
	}

	stages {
	  stage('Configure credentials') {
	    steps {
	      withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-worker-1-key', keyFileVariable: 'private_key', usernameVariable: 'username')]) {
	        script {
	          remote.name = "${env.HOST}"
	          remote.host = "${env.HOST}"
	          remote.user = "$username"
	          remote.identity = readFile "$private_key"
	          remote.allowAnyHosts = true
	        }
	      }
	    }
	  }

	  stage('Checkout repo') {
	    steps {
	      git branch: params.BRANCH, url: 'git@github.com:AnastasiyaGapochkina01/dos-34-app-01.git', credentialsId: 'jenkins-key'
	    }
	  }

	  stage('Build image') {
	    steps {
	      script {
	        sh """
              docker build -t "${env.REGISTRY}:${env.PRJ_NAME}-${BUILD_ID}" .
	        """
	      }
	    }
	  }

	  stage('Push image') {
	    steps {
	      script {
	        sh """
              docker login -u anestesia01 -p "${env.DOCKER_TOKEN}"
              docker push "${env.REGISTRY}:${env.PRJ_NAME}-${BUILD_ID}"
              docker logout
	        """
	      }
	    }
	  }

	  stage('Run tests') {
		  when {
                expression { return params.RUN_TESTS }
            }
		  steps {
			  script {
				  sh """
                    docker pull "${env.REGISTRY}:${env.PRJ_NAME}-${BUILD_ID}"
					docker run --rm ${IMAGE}:${TAG} npm test
				  """
			  }
		  }
	  }

	  stage('Deploy') {
	    steps {
	      script {
	        sshCommand remote: remote, command: """
              docker pull "${env.REGISTRY}:${env.PRJ_NAME}-${BUILD_ID}"
              docker rm "${env.PRJ_NAME}" -f || true
              docker run -d -it --name "${env.PRJ_NAME}" -p 3000:3000 "${env.REGISTRY}:${env.PRJ_NAME}-${BUILD_ID}"
	        """
	      }
	    }
	  }

	  stage('Check after deploy') {
	    steps {
	      script {
	        sh """
			  sleep 5
              curl -s -o /dev/null -w "%{http_code}" ${env.HOST}:3000
	        """
	      }
	    }
	  }
	}
}
