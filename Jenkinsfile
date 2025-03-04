pipeline {
    agent any
    environment {
        VERSION = "1.0"  
    }
    stages {
        stage('Dev') {
            steps {
                script {
                    sh "helm upgrade --install python-app /home/sampreeth/Multi-Stage-Project/python-app -n dev --set image.tag=${VERSION}"
                    def approval = input(
                        message: 'Approve or Reject deployment to Test?',
                        parameters: [choice(name: 'decision', choices: ['Approve', 'Reject'], description: 'Select your action')]
                    )
                    if (approval == 'Reject') {
                        sh "helm rollback python-app -n dev"
                        error("Dev deployment rejected! Rolling back Dev and stopping pipeline.")
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    try {
                        sh "helm upgrade --install python-app /home/sampreeth/Multi-Stage-Project/python-app -n test --set image.tag=${VERSION}"
                        def approval = input(
                            message: 'Test Team: Approve or Reject deployment to Prod?',
                            parameters: [choice(name: 'decision', choices: ['Approve', 'Reject'], description: 'Select your action')]
                        )
                        if (approval == 'Reject') {
                            sh "helm rollback python-app -n test"
                            error("Test deployment rejected! Rolling back Test and stopping pipeline.")
                        }
                    } catch (Exception e) {
                        sh "helm rollback python-app -n test"
                        error("Test deployment failed! Rolling back Test and stopping pipeline.")
                    }
                }
            }
        }

        stage('Prod') {
            steps {
                script {
                    try {
                        sh "helm upgrade --install python-app /home/sampreeth/Multi-Stage-Project/python-app -n prod --set image.tag=${VERSION}"
                    } catch (Exception e) {
                        sh "helm rollback python-app -n prod"
                        error("Prod deployment failed! Rolling back Prod and stopping pipeline.")
                    }
                }
            }
        }
    }
    post {
        success {
            script {
                def oldVersion = VERSION.toFloat()
                def newVersion = oldVersion + 0.1
                env.VERSION = newVersion.toString()
                echo "New version: ${VERSION}"
            }
        }
    }
}