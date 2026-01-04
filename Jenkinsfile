// Function to select Registry based on Parameter input
def registrySelector(environment) {
    switch (environment) {
        case "DEV" :
            return "app-dev-cloud"
        case "QA" :
            return "app-qa-cloud"
        case "STAGING" :
            return "app-staging-team"
        case "PRODUCTION" :
            return "app-prod-team"
        default :
            return "app-dev-cloud"
    }
}

pipeline {
    agent any

    parameters {
        string(defaultValue: '', description: 'Enter Git Repo Name', name: 'GITREPO', trim: true)
        gitParameter(branchFilter: 'origin/(.*)', defaultValue: 'main', name: 'GIT_BRANCHES', type: 'PT_BRANCH', description: 'Select branch', useRepository: '.*${params.GITREPO}.git')
        choice(choices: ['--None--','DEV','QA','STAGING','PRODUCTION'], description: 'Select Environment', name: 'Environment_Name')
        choice(choices: ['--None--','JDK_21','JDK_11','JDK_8'], description: 'Select JDK version', name: 'SELECT_TECH')
        choice(choices: ['--None--','Maven','Gradle'], description: 'Select build tool', name: 'BUILD_TOOL_SELECTION')
    }

    tools {
        // Dynamically select Maven tool (ensure name matches Global Tool Config)
        maven 'Maven-3'
    }

    stages {
        stage('Validation') {
            steps {
                script {
                    if (params.Environment_Name == '--None--' || params.GITREPO == '') {
                        error "Stopping Build: Environment or Repo Name not selected!"
                    }
                }
            }
        }

        stage('Git Checkout') {
            steps {
                // Uses the parameters to checkout the specific branch and repo
                git branch: "${params.GIT_BRANCHES}", url: "https://github.com/PrashantaPH/${params.GITREPO}.git"
            }
        }

        stage('Mvn Build') {
            when { expression { params.BUILD_TOOL_SELECTION == 'Maven' } }
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Deploy') {
            steps {
                script {
                    // Use the function to get registry based on the CHOICE parameter
                    def registryNamespace = registrySelector(params.Environment_Name)
                    def imageName = "${params.PRODUCT_NAME.toLowerCase()}-${params.GITREPO.toLowerCase()}"
                    def fullImageTag = "${registryNamespace}/${imageName}:${env.BUILD_NUMBER}"
                    def containerName = "${imageName}-container"

                    echo "Deploying ${params.PRODUCT_NAME} to ${params.Environment_Name} environment..."

                    sh "docker build --no-cache -t ${fullImageTag} ."
                    sh "docker rm -f ${containerName} || true"
                    sh "docker run -d --name ${containerName} -p 9091:8081 ${fullImageTag}"
                }
            }
        }
    }

    post {
        success {
            echo "Successfully deployed version ${env.BUILD_NUMBER} to ${params.Environment_Name}"
        }
    }
}
