/* pipeline ���� ���� */
def DOCKER_IMAGE_NAME = "lshyeung/project-repo"           // �����ϴ� Docker image �̸�
def DOCKER_IMAGE_TAGS = "batch-visualizer-auth"  // �����ϴ� Docker image �±�
def NAMESPACE = "ns-project"
def VERSION = "${env.BUILD_NUMBER}"
def DATE = new Date();
  
podTemplate(label: 'builder',
            containers: [
                containerTemplate(name: 'gradle', image: 'gradle:6.8.2-jdk8', command: 'cat', ttyEnabled: true),
                containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
                containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.20.7', command: 'cat', ttyEnabled: true)
            ],
            volumes: [
                hostPathVolume(mountPath: '/home/gradle/.gradle', hostPath: '/home/admin/k8s/jenkins/.gradle'),
                hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
                //hostPathVolume(mountPath: '/usr/bin/docker', hostPath: '/usr/bin/docker')
            ]) {
    node('builder') {
        stage('Checkout') {
             checkout scm   // gitlab���κ��� �ҽ� �ٿ�
        }
        stage('Build') {
            container('gradle') {
                /* ��Ŀ �̹����� Ȱ���Ͽ� gradle ���带 �����Ͽ� ./build/libs�� jar���� ���� */
                sh "gradle -x test build"
            }
        }
        stage('Docker build') {
            container('docker') {
                withCredentials([usernamePassword(
                    credentialsId: 'docker_hub_auth',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD')]) {
                        /* ./build/libs ������ jar������ ��Ŀ������ Ȱ���Ͽ� ��Ŀ ���带 �����Ѵ� */
                        sh "docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAGS} ."
                        sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                        sh "docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAGS}"
                }
            }
        }
        stage('Run kubectl') {
            container('kubectl') {
                withCredentials([usernamePassword(
                    credentialsId: 'docker_hub_auth',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD')]) {
                        /* namespace ���翩�� Ȯ��. ������� namespace ���� */
                        sh "kubectl get ns ${NAMESPACE}|| kubectl create ns ${NAMESPACE}"

                        /* secret ���翩�� Ȯ��. ������� secret ���� */
                        sh """
                            kubectl get secret my-secret -n ${NAMESPACE} || \
                            kubectl create secret docker-registry my-secret \
                            --docker-server=https://index.docker.io/v1/ \
                            --docker-username=${USERNAME} \
                            --docker-password=${PASSWORD} \
                            --docker-email=ekfrl2815@gmail.com \
                            -n ${NAMESPACE}
                        """
                        /* k8s-deployment.yaml �� env���� �������ش�(DATE��). ������ ������ ������ ������ ����� ������ ���� �������� �ʴ´�. */
                        /*sh "echo ${VERSION}"
                        sh "sed -i.bak 's#VERSION_STRING#${VERSION}#' ./k8s/k8s-deployment.yaml"*/
                        sh "echo ${DATE}"
                        sh "sed -i.bak 's#DATE_STRING#${DATE}#' ./k8s/k8s-deployment.yaml"

                        /* yaml���Ϸ� ������ �����Ѵ� */
                        sh "kubectl apply -f ./k8s/k8s-deployment.yaml -n ${NAMESPACE}"
                        sh "kubectl apply -f ./k8s/k8s-service.yaml -n ${NAMESPACE}"
                }
            }
        }
    }
}