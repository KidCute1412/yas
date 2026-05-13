import groovy.transform.Field

@Field def SERVICE_PARAM_MAP = [
    cart            : 'TAG_CART',
    tax             : 'TAG_TAX',
    order           : 'TAG_ORDER',
    product         : 'TAG_PRODUCT',
    media           : 'TAG_MEDIA',
    rating          : 'TAG_RATING',
    customer        : 'TAG_CUSTOMER',
    location        : 'TAG_LOCATION',
    inventory       : 'TAG_INVENTORY',
    search          : 'TAG_SEARCH',
    payment         : 'TAG_PAYMENT',
    promotion       : 'TAG_PROMOTION',
    recommendation  : 'TAG_RECOMMENDATION',
    sampledata      : 'TAG_SAMPLEDATA',
    webhook         : 'TAG_WEBHOOK',
    'backoffice-bff': 'TAG_BACKOFFICE_BFF',
    'storefront-bff': 'TAG_STOREFRONT_BFF',
    'backoffice-ui' : 'TAG_BACKOFFICE_UI',
    'storefront-ui' : 'TAG_STOREFRONT_UI'
]

@Field def UI_SERVICES = [
    'backoffice-ui',
    'storefront-ui'
]

@Field def SERVICE_ALIASES = [
    'backoffice-ui' : ['backoffice-ui', 'backoffice'],
    'storefront-ui' : ['storefront-ui', 'storefront']
]

@Field def DOCKERFILE_PATTERNS = [
    '%s/Dockerfile',
    'services/%s/Dockerfile',
    'src/%s/Dockerfile',
    'apps/%s/Dockerfile',
    'applications/%s/Dockerfile'
]

def resolveServicesToBuild() {
    if (env.BRANCH_NAME == env.MAIN_BRANCH) {
        echo "Main branch detected. Build all services."
        return SERVICE_PARAM_MAP.keySet() as List
    }

    def branchShortName = env.BRANCH_NAME.tokenize('/').last()
    def matcher = branchShortName =~ /^dev_(.+)_service$/

    if (!matcher.matches()) {
        error """
Unsupported branch name: ${env.BRANCH_NAME}

For non-main branches, use this naming convention:
  dev_<service>_service

Examples:
  dev_tax_service
  dev_cart_service
  dev_product_service
  dev_storefront_bff_service
  dev_backoffice_ui_service
"""
    }

    def serviceFromBranch = matcher[0][1].replace('_', '-')

    if (!SERVICE_PARAM_MAP.containsKey(serviceFromBranch)) {
        error """
Cannot map branch '${env.BRANCH_NAME}' to a known service.

Detected service:
  ${serviceFromBranch}

Known services:
  ${SERVICE_PARAM_MAP.keySet().join(', ')}
"""
    }

    echo "Branch '${env.BRANCH_NAME}' maps to service '${serviceFromBranch}'."
    return [serviceFromBranch]
}

def findDockerConfig(String serviceName) {
    def candidateNames = SERVICE_ALIASES.containsKey(serviceName)
        ? SERVICE_ALIASES[serviceName]
        : [serviceName]

    def dockerConfig = null

    candidateNames.each { candidateName ->
        DOCKERFILE_PATTERNS.each { pattern ->
            if (dockerConfig == null) {
                def dockerfilePath = String.format(pattern, candidateName)

                if (fileExists(dockerfilePath)) {
                    def contextPath = dockerfilePath.substring(0, dockerfilePath.lastIndexOf('/'))

                    dockerConfig = [
                        dockerfile: dockerfilePath,
                        context   : contextPath
                    ]
                }
            }
        }
    }

    if (dockerConfig == null) {
        error """
Cannot find Dockerfile for service: ${serviceName}

Checked names:
${candidateNames}

Checked patterns:
${DOCKERFILE_PATTERNS}

Please update SERVICE_ALIASES or DOCKERFILE_PATTERNS in Jenkinsfile.
"""
    }

    return dockerConfig
}

pipeline {
    agent { label 'build-agent' }

    options {
        timestamps()
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
    }

    environment {
        DOCKERHUB_NAMESPACE = 'thanhtienntt'
        DOCKERHUB_CREDS = 'dockerhub-creds'
        MAIN_BRANCH = 'main'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Image Tag and Services') {
            steps {
                script {
                    env.COMMIT_ID = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()

                    if (env.BRANCH_NAME == env.MAIN_BRANCH) {
                        env.IMAGE_TAG = 'latest'
                    } else {
                        env.IMAGE_TAG = env.COMMIT_ID
                    }

                    def servicesToBuild = resolveServicesToBuild()
                    env.SERVICES_TO_BUILD = servicesToBuild.join(',')

                    echo "Branch name       : ${env.BRANCH_NAME}"
                    echo "Commit id         : ${env.COMMIT_ID}"
                    echo "Image tag         : ${env.IMAGE_TAG}"
                    echo "Services to build : ${env.SERVICES_TO_BUILD}"
                }
            }
        }

        stage('Check Build Tools') {
            steps {
                sh '''
                    echo "===== Build Agent Info ====="
                    whoami
                    pwd

                    echo "===== Git ====="
                    git --version
                    git log --oneline -1

                    echo "===== Java ====="
                    java -version || true
                    javac -version || true

                    echo "===== Maven ====="
                    mvn -version || true

                    echo "===== Docker ====="
                    docker version
                '''
            }
        }

        stage('Build Java Services') {
            steps {
                script {
                    def servicesToBuild = env.SERVICES_TO_BUILD.split(',') as List

                    def javaServices = servicesToBuild.findAll { serviceName ->
                        !UI_SERVICES.contains(serviceName)
                    }

                    if (javaServices.isEmpty()) {
                        echo "No Java services need Maven build."
                        return
                    }

                    def modules = javaServices.join(',')

                    sh """
                        echo "===== Build Java modules ====="
                        echo "Modules: ${modules}"

                        if [ -f "./mvnw" ]; then
                            chmod +x ./mvnw
                            ./mvnw -B -ntp -pl ${modules} -am clean package -DskipTests
                        else
                            mvn -B -ntp -pl ${modules} -am clean package -DskipTests
                        fi

                        echo "===== Generated JAR files ====="
                        find . -path "*/target/*.jar" -type f | head -100
                    """
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKERHUB_CREDS}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Build and Push Selected Services') {
            steps {
                script {
                    def servicesToBuild = env.SERVICES_TO_BUILD.split(',') as List

                    servicesToBuild.each { serviceName ->
                        def dockerConfig = findDockerConfig(serviceName)
                        def imageName = "${DOCKERHUB_NAMESPACE}/yas-${serviceName}:${IMAGE_TAG}"

                        sh """
                            echo "=========================================="
                            echo "Building service : ${serviceName}"
                            echo "Image            : ${imageName}"
                            echo "Dockerfile       : ${dockerConfig.dockerfile}"
                            echo "Context          : ${dockerConfig.context}"
                            echo "=========================================="

                            if [ -d "${dockerConfig.context}/target" ]; then
                                echo "Files in target:"
                                ls -la "${dockerConfig.context}/target"
                            fi

                            docker build \
                              -f "${dockerConfig.dockerfile}" \
                              -t "${imageName}" \
                              "${dockerConfig.context}"

                            docker push "${imageName}"
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }

        success {
            echo "Images were built and pushed successfully."
            echo "Tag: ${env.IMAGE_TAG}"
            echo "Services: ${env.SERVICES_TO_BUILD}"
        }

        failure {
            echo "Pipeline failed. Check branch naming, Java version, Maven build, Dockerfile paths, Docker daemon, or Docker Hub credentials."
        }
    }
}