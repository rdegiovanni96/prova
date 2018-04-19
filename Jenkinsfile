pipeline {
    agent none
	
    stages {
        stage('Build') {
            agent  {
                label 'master'
            }
			environment {
				PATH = "${tool 'ant-1.10.1'}/bin:${PATH}"
			}
            steps {
				echo 'Build on Build environment'
				checkout scm
				sh 'ant -f build.xml'
				junit "build/test/TEST*.xml"
				sh 'cp build/helloworld-0.1-dev.war /vagrant/artifact.repo/helloworld'
            }
        }
		stage ('Promotion to Test') {
		    agent  {
                label 'master'
            }
			steps {
				timeout(time: 1, unit: 'HOURS') {
					input 'Eseguire il deployment in Test?'
				}
			}
		}
        stage('Test on Test Machine') {
            agent {
                label 'test'
            }
            steps {
                echo 'Seleniun test on TEST environment'
				sh 'sudo cp /vagrant/artifact.repo/helloworld/helloworld-0.1-dev.war /var/lib/tomcat7/webapps'
            }
        }
		stage('Provisioning Tomcat on QA Machine') {
            agent {
                label 'master'
            }
            steps {
                echo 'Provisioning Tomcat on QA'
				ansiblePlaybook credentialsId: 'vagrant', installation: 'ansible-2.3.1.0', inventory: './playbook/hosts/hosts.qa', playbook: './playbook/site.yml', sudoUser: null
            }
        }
		stage ('Promotion to QA') {
		    agent  {
                label 'master'
            }
			steps {
				timeout(time: 1, unit: 'HOURS') {
					input 'Eseguire il deployment in QA?'
				}
			}
		}
		stage('Test on QA Machine') {
            agent {
                label 'qa'
            }
            steps {
                echo 'Seleniun test on QA environment'
				sh 'sudo cp /vagrant/artifact.repo/helloworld/helloworld-0.1-dev.war /opt/apache-tomcat-7.0.61/webapps'
            }
        }
		stage('Docker packaging and publishing on Docker Registry') {
            agent {
                label 'docker'
            }
            steps {
                echo 'Preparing and uploading new Image'
				sh '''sudo cp /vagrant/artifact.repo/helloworld/helloworld-0.1-dev.war .
					sudo docker pull tomcat:7.0.82-jre8
					sudo docker build -t helloworld .
					sudo docker tag helloworld 192.168.56.18:5000/helloworld
					sudo docker push 192.168.56.18:5000/helloworld
					sudo docker rmi 192.168.56.18:5000/helloworld
					sudo docker rmi helloworld'''
            }
        }
		stage('Deploy to Production by Docker Pull') {
            agent {
                label 'prod'
            }
            steps {
                echo 'Seleniun test on QA environment'
				sh '''if [ "$(sudo docker ps -q -f name=helloworld)" ]; then 
					sudo docker rm -f helloworld
					sudo docker rmi 192.168.56.18:5000/helloworld
				fi
				# pull your image
				sudo docker pull 192.168.56.18:5000/helloworld
				# run your container
				sudo docker run -d -it --name helloworld --rm -p 8080:8080 192.168.56.18:5000/helloworld'''
            }
        }
    }
}
