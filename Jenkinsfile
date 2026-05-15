// import groovy.transform.Field

// @Field def SERVICE_PARAM_MAP = [
//     cart            : 'TAG_CART',
//     tax             : 'TAG_TAX',
//     order           : 'TAG_ORDER',
//     product         : 'TAG_PRODUCT',
//     media           : 'TAG_MEDIA',
//     rating          : 'TAG_RATING',
//     customer        : 'TAG_CUSTOMER',
//     location        : 'TAG_LOCATION',
//     inventory       : 'TAG_INVENTORY',
//     search          : 'TAG_SEARCH',
//     payment         : 'TAG_PAYMENT',
//     promotion       : 'TAG_PROMOTION',
//     recommendation  : 'TAG_RECOMMENDATION',
//     sampledata      : 'TAG_SAMPLEDATA',
//     webhook         : 'TAG_WEBHOOK',
//     'backoffice-bff': 'TAG_BACKOFFICE_BFF',
//     'storefront-bff': 'TAG_STOREFRONT_BFF',
//     'backoffice-ui' : 'TAG_BACKOFFICE_UI',
//     'storefront-ui' : 'TAG_STOREFRONT_UI'
// ]

// @Field def UI_SERVICES = [
//     'backoffice-ui',
//     'storefront-ui'
// ]

// @Field def SERVICE_ALIASES = [
//     'backoffice-ui' : ['backoffice-ui', 'backoffice'],
//     'storefront-ui' : ['storefront-ui', 'storefront']
// ]

// @Field def DOCKERFILE_PATTERNS = [
//     '%s/Dockerfile',
//     'services/%s/Dockerfile',
//     'src/%s/Dockerfile',
//     'apps/%s/Dockerfile',
//     'applications/%s/Dockerfile'
// ]

// def resolveServicesToBuild() {
//     if (env.BRANCH_NAME == env.MAIN_BRANCH) {
//         echo "Main branch detected. Build all services."
//         return SERVICE_PARAM_MAP.keySet() as List
//     }

//     def branchShortName = env.BRANCH_NAME.tokenize('/').last()
//     def matcher = branchShortName =~ /^dev_(.+)_service$/

//     if (!matcher.matches()) {
//         error """
// Unsupported branch name: ${env.BRANCH_NAME}

// For non-main branches, use this naming convention:
//   dev_<service>_service

// Examples:
//   dev_tax_service
//   dev_cart_service
//   dev_product_service
//   dev_storefront_bff_service
//   dev_backoffice_ui_service
// """
//     }

//     def serviceFromBranch = matcher[0][1].replace('_', '-')

//     if (!SERVICE_PARAM_MAP.containsKey(serviceFromBranch)) {
//         error """
// Cannot map branch '${env.BRANCH_NAME}' to a known service.

// Detected service:
//   ${serviceFromBranch}

// Known services:
//   ${SERVICE_PARAM_MAP.keySet().join(', ')}
// """
//     }

//     echo "Branch '${env.BRANCH_NAME}' maps to service '${serviceFromBranch}'."
//     return [serviceFromBranch]
// }

// def findDockerConfig(String serviceName) {
//     def candidateNames = SERVICE_ALIASES.containsKey(serviceName)
//         ? SERVICE_ALIASES[serviceName]
//         : [serviceName]

//     def dockerConfig = null

//     candidateNames.each { candidateName ->
//         DOCKERFILE_PATTERNS.each { pattern ->
//             if (dockerConfig == null) {
//                 def dockerfilePath = String.format(pattern, candidateName)

//                 if (fileExists(dockerfilePath)) {
//                     def contextPath = dockerfilePath.substring(0, dockerfilePath.lastIndexOf('/'))

//                     dockerConfig = [
//                         dockerfile: dockerfilePath,
//                         context   : contextPath
//                     ]
//                 }
//             }
//         }
//     }

//     if (dockerConfig == null) {
//         error """
// Cannot find Dockerfile for service: ${serviceName}

// Checked names:
// ${candidateNames}

// Checked patterns:
// ${DOCKERFILE_PATTERNS}

// Please update SERVICE_ALIASES or DOCKERFILE_PATTERNS in Jenkinsfile.
// """
//     }

//     return dockerConfig
// }

// pipeline {
//     agent { label 'build-agent' }

//     options {
//         timestamps()
//         disableConcurrentBuilds()
//         skipDefaultCheckout(true)
//     }

//     environment {
//         DOCKERHUB_NAMESPACE = 'thanhtienntt'
//         DOCKERHUB_CREDS = 'dockerhub-creds'
//         MAIN_BRANCH = 'main'
//     }

//     stages {
//         stage('Checkout') {
//             steps {
//                 checkout scm
//             }
//         }

//         stage('Prepare Image Tag and Services') {
//             steps {
//                 script {
//                     env.COMMIT_ID = sh(
//                         script: 'git rev-parse --short HEAD',
//                         returnStdout: true
//                     ).trim()

//                     if (env.BRANCH_NAME == env.MAIN_BRANCH) {
//                         env.IMAGE_TAG = 'latest'
//                     } else {
//                         env.IMAGE_TAG = env.COMMIT_ID
//                     }

//                     def servicesToBuild = resolveServicesToBuild()
//                     env.SERVICES_TO_BUILD = servicesToBuild.join(',')

//                     echo "Branch name       : ${env.BRANCH_NAME}"
//                     echo "Commit id         : ${env.COMMIT_ID}"
//                     echo "Image tag         : ${env.IMAGE_TAG}"
//                     echo "Services to build : ${env.SERVICES_TO_BUILD}"
//                 }
//             }
//         }

//         stage('Check Build Tools') {
//             steps {
//                 sh '''
//                     echo "===== Build Agent Info ====="
//                     whoami
//                     pwd

//                     echo "===== Git ====="
//                     git --version
//                     git log --oneline -1

//                     echo "===== Java ====="
//                     java -version || true
//                     javac -version || true

//                     echo "===== Maven ====="
//                     mvn -version || true

//                     echo "===== Docker ====="
//                     docker version
//                 '''
//             }
//         }

//         stage('Build Java Services') {
//             steps {
//                 script {
//                     def servicesToBuild = env.SERVICES_TO_BUILD.split(',') as List

//                     def javaServices = servicesToBuild.findAll { serviceName ->
//                         !UI_SERVICES.contains(serviceName)
//                     }

//                     if (javaServices.isEmpty()) {
//                         echo "No Java services need Maven build."
//                         return
//                     }

//                     def modules = javaServices.join(',')

//                     sh """
//                         echo "===== Build Java modules ====="
//                         echo "Modules: ${modules}"

//                         if [ -f "./mvnw" ]; then
//                             chmod +x ./mvnw
//                             ./mvnw -B -ntp -pl ${modules} -am clean package -DskipTests
//                         else
//                             mvn -B -ntp -pl ${modules} -am clean package -DskipTests
//                         fi

//                         echo "===== Generated JAR files ====="
//                         find . -path "*/target/*.jar" -type f | head -100
//                     """
//                 }
//             }
//         }

//         stage('Docker Login') {
//             steps {
//                 withCredentials([
//                     usernamePassword(
//                         credentialsId: "${DOCKERHUB_CREDS}",
//                         usernameVariable: 'DOCKER_USER',
//                         passwordVariable: 'DOCKER_PASS'
//                     )
//                 ]) {
//                     sh '''
//                         echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
//                     '''
//                 }
//             }
//         }

//         stage('Build and Push Selected Services') {
//             steps {
//                 script {
//                     def servicesToBuild = env.SERVICES_TO_BUILD.split(',') as List

//                     servicesToBuild.each { serviceName ->
//                         def dockerConfig = findDockerConfig(serviceName)
//                         def imageName = "${DOCKERHUB_NAMESPACE}/yas-${serviceName}:${IMAGE_TAG}"

//                         sh """
//                             echo "=========================================="
//                             echo "Building service : ${serviceName}"
//                             echo "Image            : ${imageName}"
//                             echo "Dockerfile       : ${dockerConfig.dockerfile}"
//                             echo "Context          : ${dockerConfig.context}"
//                             echo "=========================================="

//                             if [ -d "${dockerConfig.context}/target" ]; then
//                                 echo "Files in target:"
//                                 ls -la "${dockerConfig.context}/target"
//                             fi

//                             docker build \
//                               -f "${dockerConfig.dockerfile}" \
//                               -t "${imageName}" \
//                               "${dockerConfig.context}"

//                             docker push "${imageName}"
//                         """
//                     }
//                 }
//             }
//         }
//     }

//     post {
//         always {
//             sh 'docker logout || true'
//         }

//         success {
//             echo "Images were built and pushed successfully."
//             echo "Tag: ${env.IMAGE_TAG}"
//             echo "Services: ${env.SERVICES_TO_BUILD}"
//         }

//         failure {
//             echo "Pipeline failed. Check branch naming, Java version, Maven build, Dockerfile paths, Docker daemon, or Docker Hub credentials."
//         }
//     }
// }




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

def isReleaseTagBuild() {
    return env.TAG_NAME != null && env.TAG_NAME.trim()
}

def resolveServicesToBuild() {
    if (isReleaseTagBuild()) {
        echo "Release tag detected: ${env.TAG_NAME}. Build all services for staging."
        return SERVICE_PARAM_MAP.keySet() as List
    }

    if (env.BRANCH_NAME == env.MAIN_BRANCH) {
        echo "Main branch detected. Build all services for dev."
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

        GITOPS_REPO_URL = 'https://github.com/KidCute1412/yas-gitops.git'
        GITOPS_BRANCH = 'main'
        GITOPS_CREDS = 'github-gitops-token'
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

                    env.EXTRA_IMAGE_TAG = ''
                    env.GITOPS_ENV = ''

                    if (isReleaseTagBuild()) {
                        env.IMAGE_TAG = env.TAG_NAME
                        env.GITOPS_ENV = 'staging'
                    } else if (env.BRANCH_NAME == env.MAIN_BRANCH) {
                        env.IMAGE_TAG = env.COMMIT_ID
                        env.EXTRA_IMAGE_TAG = 'latest'
                        env.GITOPS_ENV = 'dev'
                    } else {
                        env.IMAGE_TAG = env.COMMIT_ID
                    }

                    def servicesToBuild = resolveServicesToBuild()
                    env.SERVICES_TO_BUILD = servicesToBuild.join(',')

                    echo "Branch name       : ${env.BRANCH_NAME}"
                    echo "Tag name          : ${env.TAG_NAME ?: 'N/A'}"
                    echo "Commit id         : ${env.COMMIT_ID}"
                    echo "Image tag         : ${env.IMAGE_TAG}"
                    echo "Extra image tag   : ${env.EXTRA_IMAGE_TAG ?: 'N/A'}"
                    echo "GitOps env        : ${env.GITOPS_ENV ?: 'N/A'}"
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
                        def imageBase = "${DOCKERHUB_NAMESPACE}/yas-${serviceName}"

                        def tagsToPush = [env.IMAGE_TAG]

                        if (env.EXTRA_IMAGE_TAG != null && env.EXTRA_IMAGE_TAG.trim()) {
                            tagsToPush.add(env.EXTRA_IMAGE_TAG)
                        }

                        def dockerTagArgs = tagsToPush.collect { tag ->
                            "-t \"${imageBase}:${tag}\""
                        }.join(' ')

                        def dockerPushCommands = tagsToPush.collect { tag ->
                            "docker push \"${imageBase}:${tag}\""
                        }.join('\n')

                        sh """
                            echo "=========================================="
                            echo "Building service : ${serviceName}"
                            echo "Image base       : ${imageBase}"
                            echo "Tags             : ${tagsToPush.join(', ')}"
                            echo "Dockerfile       : ${dockerConfig.dockerfile}"
                            echo "Context          : ${dockerConfig.context}"
                            echo "=========================================="

                            if [ -d "${dockerConfig.context}/target" ]; then
                                echo "Files in target:"
                                ls -la "${dockerConfig.context}/target"
                            fi

                            docker build \
                              -f "${dockerConfig.dockerfile}" \
                              ${dockerTagArgs} \
                              "${dockerConfig.context}"

                            ${dockerPushCommands}
                        """
                    }
                }
            }
        }

        stage('Update GitOps Repo') {
            when {
                expression {
                    return env.GITOPS_ENV != null && env.GITOPS_ENV.trim()
                }
            }

            agent { label 'deploy-agent' }

            steps {
                script {
                    def servicesToBuild = env.SERVICES_TO_BUILD.split(',') as List
                    def targetValuesFile = "envs/${env.GITOPS_ENV}/values.yaml"

                    def yqCommands = servicesToBuild.collect { serviceName ->
                        def repository = "${env.DOCKERHUB_NAMESPACE}/yas-${serviceName}"

                        return """
                            yq -i '.services."${serviceName}".image.repository = "${repository}"' ${targetValuesFile}
                            yq -i '.services."${serviceName}".image.tag = "${env.IMAGE_TAG}"' ${targetValuesFile}
                        """
                    }.join('\n')

                    def gitopsRepoWithoutProtocol = env.GITOPS_REPO_URL.replaceFirst('https://', '').replaceFirst('http://', '')

                    withCredentials([
                        usernamePassword(
                            credentialsId: "${GITOPS_CREDS}",
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_TOKEN'
                        )
                    ]) {
                        sh """
                            set -e

                            echo "===== Deploy Agent Info ====="
                            whoami
                            pwd

                            echo "===== Check tools ====="
                            git --version
                            yq --version

                            echo "===== Clone GitOps repo ====="
                            rm -rf yas-gitops

                            git clone -b "${GITOPS_BRANCH}" "https://\\${GIT_USERNAME}:\\${GIT_TOKEN}@${gitopsRepoWithoutProtocol}" yas-gitops

                            cd yas-gitops

                            git config user.name "jenkins-bot"
                            git config user.email "jenkins-bot@example.com"

                            echo "===== Update GitOps values ====="
                            echo "Environment      : ${env.GITOPS_ENV}"
                            echo "Values file      : ${targetValuesFile}"
                            echo "Image tag        : ${env.IMAGE_TAG}"
                            echo "Services         : ${env.SERVICES_TO_BUILD}"

                            if [ ! -f "${targetValuesFile}" ]; then
                                echo "Missing GitOps values file: ${targetValuesFile}"
                                exit 1
                            fi

                            ${yqCommands}

                            echo "===== Git diff ====="
                            git diff

                            git add "${targetValuesFile}"

                            if git diff --cached --quiet; then
                                echo "No GitOps changes to commit."
                            else
                                git commit -m "chore(${env.GITOPS_ENV}): update YAS images to ${env.IMAGE_TAG}"
                                git push origin "${GITOPS_BRANCH}"
                            fi
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
            echo "Primary tag: ${env.IMAGE_TAG}"
            echo "Extra tag: ${env.EXTRA_IMAGE_TAG ?: 'N/A'}"
            echo "Services: ${env.SERVICES_TO_BUILD}"
            echo "GitOps env: ${env.GITOPS_ENV ?: 'N/A'}"
        }

        failure {
            echo "Pipeline failed. Check branch naming, Java version, Maven build, Dockerfile paths, Docker daemon, Docker Hub credentials, GitOps credentials, yq, or GitOps values path."
        }
    }
}