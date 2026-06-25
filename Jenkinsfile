pipeline{
    agent any
    parameters{
        booleanParam(name:'ALLURE', defaultValue: false, description: 'generation de rapport allure')
    }
    stages{
        stage('global stage'){
            agent{
                docker{
                    image 'postman/newman:latest'
                    args '-u root --entrypoint='
                }
            }
            stages{
                stage('install deps'){
                    steps{
                        sh 'npm ci'
                    }
                }

                stage('clean allure results'){
                    steps{
                        sh '''
                            echo "Suppression du cache Allure..."
                            rm -rf allure-results
                            mkdir -p allure-results
                            echo "Dossier allure-results nettoyé avec succès"
                        '''
                    }
                }
        
                stage('run user test'){
                    steps{  
                        script{
                            if(params.ALLURE){
                                sh '''
                                    newman run collection.json -r allure --reporter-allure-export allure-results
                                '''
                            } else {
                                sh '''
                                    newman run collection.json
                                '''
                            }
                        }
                    }
                }

                stage('stash allure results'){
                    when{
                        expression { params.ALLURE }
                    }
                    steps{
                        stash includes: 'allure-results/**', name: 'allure-results'
                    }
                }
            }
        }
    }
    post{
        always{
            script{
                if(params.ALLURE){
                    unstash 'allure-results'
                    archiveArtifacts 'allure-results/*'
                    allure includeProperties: false,
                           jdk: '',
                           results: [[path: 'allure-results/']]
                }
            }
        }
    }
}