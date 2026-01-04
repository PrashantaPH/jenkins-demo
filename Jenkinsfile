def registrySelector(environment) {
    switch (environment) {
        case "DEV" : return "app-dev-cloud"
        case "QA" : return "app-qa-cloud"
        case "STAGING" : return "app-staging-team"
        case "PRODUCTION" : return "app-prod-team"
        default : return "app-dev-cloud"
    }
}

pipeline {
    agent any

    parameters {
        string(defaultValue: '', description: 'Enter Git Repo Name', name: 'GITREPO', trim: true)
        // Note: Requires "Git Parameter" Plugin installed
        gitParameter(branchFilter: 'origin/(.*)', defaultValue: 'main', name: 'GIT_BRANCHES', type: 'PT_BRANCH', description: 'Select branch', useRepository: '.*${params.GITREPO}.git')
        choice(choices: ['--None--','DEV','QA','STAGING','PRODUCTION'], description: 'Select Environment', name: 'Environment_Name')
        // Added this back because your Docker stage uses it
        choice(choices: ['--None--','PLATFORM','DSS','XPONENT','IDX'], description: 'Select Product', name: 'PRODUCT_NAME')
        choice(choices: ['--None--','JDK_21','JDK_11','JDK_8'], description: 'Select JDK version', name: 'SELECT_TECH')
        choice(choices: ['--None--','Maven','Gradle'], description: 'Select build tool', name: 'BUILD_TOOL_SELECTION')
    }

    tools {
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
                // IMPORTANT: Ensure your Git credentials are set in Jenkins if repo is private
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
                    def registryNamespace = registrySelector(params.Environment_Name)
                    // sanitize name (lowercase and no spaces)
                    def prodName = params.PRODUCT_NAME.toLowerCase().replace('--none--', 'app')
                    def repoName = params.GITREPO.toLowerCase()
                    
                    def imageName = "${prodName}-${repoName}"
                    def fullImageTag = "${registryNamespace}/${imageName}:${env.BUILD_NUMBER}"
                    def containerName = "${imageName}-container"

                    echo "Deploying ${prodName} to ${params.Environment_Name}..."

                    sh "docker build --no-cache -t ${fullImageTag} ."
                    sh "docker rm -f ${containerName} || true"
                    // Mapping to 9091 as per your request
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
