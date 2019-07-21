/*
如果存在用户输入，则使用用户输入image_tag,否则${git_commit}为image_tag
${DOCKER_TAG} -- 155ada5a2aa90c992afd16413fabe4438c0e11bb

IMAGE_NAME为完整image地址
${IMAGE_NAME} -- harbor.qyvideo.net/duanyifei-test/centos:155ada5a2aa90c992afd16413fabe4438c0e11bb

HELM_TAG 始终为${GIT_COMMIT}, 与chart.yaml对应；
values.yaml 为image_tag, 实现配置与image解耦
${HELM_TAG}   -- 155ada5a2aa90c992afd16413fabe4438c0e11bb
*/
def k8s_deploy (HOST, NAMESPACE) {
    def remote = [:]
    remote.name = 'k8s-master'
    remote.host = "${HOST}"
    remote.user = "${env.SSH_REMOTE_USER}"
    remote.password = "${env.SSH_REMOTE_PWD}"
    remote.allowAnyHosts = true
    stage('Remote SSH') {
        sshCommand remote: remote, command: "pwd"
        sshCommand remote: remote, command: """
        helm upgrade "${APP_NAME}-${NAMESPACE}" --debug --install --namespace="${NAMESPACE}" --set "global.env=${NAMESPACE}" "${env.HELM_REPO}/charts/${APP_NAME}-${GIT_COMMIT}.tgz"
      """
    }
}


def docker_build (IMAGE_NAME) {
    echo "docker build ~~ ${IMAGE_NAME}"

    script {
        echo "docker build"
        sh "sudo docker build -f docker/Dockerfile -t ${IMAGE_NAME} ."
    }
}


def docker_image_push (IMAGE_NAME) {
    echo "Push docker image to harbor ~~ ${IMAGE_NAME}"
    // push docker images
    echo "docker registry login"
    sh "sudo docker login -u ${env.HARBOR_USER} -p ${env.HARBOR_PSW} ${env.HARBOR_REG}"

    echo "Pushing ${IMAGE_NAME} image to registry"
    sh "sudo docker push ${IMAGE_NAME}"
}


// 改写helm values.yaml:image.tag & chart.yaml:version 为当前docker-tag
def helm_chart_push (DOCKER_TAG, HELM_TAG) {
    echo "Push docker image to harbor ~~ ${DOCKER_TAG}"
        // modify helm version and push chartmuseum
    script {
        def filename = 'helm/values.yaml'
        def data = readYaml file: filename
        echo "${DOCKER_TAG}"
        data.image.tag = "${DOCKER_TAG}"
        sh "rm $filename"
        writeYaml file: filename, data: data
    }

    sh 'cat helm/values.yaml'

    script {
        def filename = 'helm/Chart.yaml'
        def data = readYaml file: filename
        data.version = "${HELM_TAG}"
        sh "rm $filename"
        writeYaml file: filename, data: data
    }

    sh 'cat helm/Chart.yaml'

    dir('helm') {
        sh "sudo /usr/local/bin/helm push . ${env.HELM_REPO} --version=${HELM_TAG} --force"
    }
}

pipeline {
    agent any

    environment {
        APP_NAME = 'hao123'
        PROJECT = 'duanyifei-test'
        // SERVICE_PORT = 8511
        // TEST_LOCAL_PORT = 8899
    }

    parameters {
        string (name: 'DOCKER_TAG', defaultValue: '', description: 'Already exist docker image~')
    }

    stages {

        ////////// Step 2 docker-build //////////
        // 当用户输入为空，需要重新docker-build
        stage('No exist docker image') {
            when {
                environment name: 'DOCKER_TAG', value: ''
            }
            steps {
                script {
                    // // 用当前GIT_COMMIT 替换 DOCKER_TAG and HELM_TAG
                    DOCKER_TAG = "${GIT_COMMIT}"
                    HELM_TAG = "${GIT_COMMIT}"
                    IMAGE_NAME = "${env.HARBOR_REG}/${PROJECT}/${APP_NAME}:${DOCKER_TAG}"
                    sh "echo ${IMAGE_NAME}"
                    docker_build (IMAGE_NAME)
                    docker_image_push (IMAGE_NAME)
                    // helm_chart_push (DOCKER_TAG, HELM_TAG)
                }
            }
        }

        // 当用户输入为空，需要重新docker-build
        stage('helm push') {
            steps {
                script {
                    // DOCKER_TAG 为用户输入 DOCKER_TAG
                    // HELM_TAG 为当 GIT_COMMIT
                    // 不需要 docker-build and docker-push, 只进行helm-push
                    DOCKER_TAG = "${DOCKER_TAG}"
                    HELM_TAG = "${GIT_COMMIT}"
                    helm_chart_push (DOCKER_TAG, HELM_TAG)
                }
            }
        }

        ////////// Step 4 //////////
        stage('Deploy to dev') {
            steps {
                script {
                    host = "${env.OFFLINE_K8S_MASTER}"
                    namespace = "${env.DEV_NAMESPACE}"
                    k8s_deploy (host, namespace)
                }
            }
        }
    }
}