pipeline {
    environment {
        LOCATION = "us-central1"
        PROJECT_ID = "devops-practice-277006"
		CLUSTER_NAME = 'cadent-jenkins-poc-cluster'
        ACCOUNT = '809054428464-compute@developer.gserviceaccount.com'
    }
    agent {
	  kubernetes {
		  label 'slave'
		  defaultContainer 'jnlp'
		  yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins/kube-default: true
    app: jenkins
    component: agent
spec:
  volumes:
  - name: sharedvolume
    emptyDir: {}
  - name: docker-socket
    emptyDir: {}
  containers:
  - name: gcloud
    image: google/cloud-sdk:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - name: sharedvolume
      mountPath: /root/.docker
    - name: docker-socket
      mountPath: /var/run
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
  - name: docker
    image: docker:19.03.1
    command:
    - sleep
    args:
    - 99d
    volumeMounts:
    - name: docker-socket
      mountPath: /var/run
    - name: sharedvolume
      mountPath: /root/.docker  
  - name: docker-daemon
    image: docker:19.03.1-dind
    securityContext:
      privileged: true
    volumeMounts:
    - name: docker-socket
      mountPath: /var/run
    - name: sharedvolume
      mountPath: /root/.docker 
  serviceAccountName: "cd-jenkins"    
  nodeSelector:
    jk_role: slave
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: jk_role
            operator: In
            values:
            - slave
  tolerations:
  - key: "jk_role"
    operator: "Equal"
    value: "slave"
    effect: "NoSchedule"
'''
      }
	}  
	  
  stages{
		stage("git checkout"){
      steps{
        script{
      git(
        url: 'https://github.com/sandeshtamboli123/gcp.git',
        credentialsId: 'github-pat-sandesh-vault',
        branch: 'main'
        )
        }
      } 
   }
        
        stage("gcloud with docker configure"){
			steps{
			  container ('gcloud') {
				script{
                    withCredentials([file(credentialsId: 'jenkins-sa-devops-practice-secretfile', variable: 'GC_KEY')]) {
                      sh("gcloud auth activate-service-account --key-file=${GC_KEY}")
					  sh("gcloud auth configure-docker")
					  sh 'docker build -t gcr.io/${PROJECT_ID}/cd-jk-upgrade/sample-app .'
					  sh 'docker push gcr.io/${PROJECT_ID}/cd-jk-upgrade/sample-app'
			       }
                }
              } 
		    }			
		}	
        stage("Application Deployment on Google Kubernetes Engine"){
            steps{
			  container ('gcloud') {
                script{
					sh "gcloud container clusters get-credentials cadent-jenkins-poc-cluster --region ${LOCATION} --project ${PROJECT_ID}"
                    sh 'kubectl apply -f deployment.yaml'
                }
              }
            }
     	}
    }
}
