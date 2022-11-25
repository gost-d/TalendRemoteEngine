node {

    stage('Checkout') {
        checkout scm
    }

    stage('Copy Talend Remote Engine files') {

        withCredentials([usernamePassword(credentialsId: 'nexus_user', usernameVariable: 'nexus_username', passwordVariable: 'nexus_password')]) {
            sh('curl -u "admin:admin123" -o "./Talend-RemoteEngine-V${tre_version}.zip" "http://172.22.6.131:8081/repository/devops/talend_remote_engine/v/${tre_version}/v-${tre_version}.zip" ')
        }
    }

    stage('Create infrastructure') {
        withCredentials([azureServicePrincipal('AzureJenkins')]) {

            sh('docker login -u admin -p admin123 http://172.22.6.131:8083')
            sh('docker pull 172.22.6.131:8083/devops/ansible:1.0')

            if (params.ostype == 'windows') {
                sh('docker run -v /home/jenkins/jenkins_home/workspace/Talend-Remote-Engine-Windows:/home/ansible --env AZURE_SUBSCRIPTION_ID=$AZURE_SUBSCRIPTION_ID --env AZURE_CLIENT_ID=$AZURE_CLIENT_ID --env AZURE_SECRET=$AZURE_CLIENT_SECRET --env AZURE_TENANT=$AZURE_TENANT_ID --rm 172.22.6.131:8083/devops/ansible:1.0 ansible-playbook /home/ansible/${ostype}/createVM.yaml')
            } else {
                sh('echo "yes" | ssh-keygen -q -t rsa -N "" -f ./id_rsa')
                sh('docker run -v /home/jenkins/jenkins_home/workspace/Talend-Remote-Engine-Windows:/home/ansible --env AZURE_SUBSCRIPTION_ID=$AZURE_SUBSCRIPTION_ID --env AZURE_CLIENT_ID=$AZURE_CLIENT_ID --env AZURE_SECRET=$AZURE_CLIENT_SECRET --env AZURE_TENANT=$AZURE_TENANT_ID --rm 172.22.6.131:8083/devops/ansible:1.0 ansible-playbook /home/ansible/${ostype}/createVM.yaml')

            }

        }
    }

    stage('Install Talend Remote Engine') {
        withCredentials([azureServicePrincipal('AzureJenkins')]) {
            if (params.ostype == 'windows') {
                 sh('docker run -v /home/jenkins/jenkins_home/workspace/Talend-Remote-Engine-Windows:/home/ansible --env AZURE_SUBSCRIPTION_ID=$AZURE_SUBSCRIPTION_ID --env AZURE_CLIENT_ID=$AZURE_CLIENT_ID --env AZURE_SECRET=$AZURE_CLIENT_SECRET --env AZURE_TENANT=$AZURE_TENANT_ID --env ANSIBLE_HOST_KEY_CHECKING=False --rm 172.22.6.131:8083/devops/ansible:1.0 ansible-playbook -vvv -i /home/ansible/${ostype}/azure_rm.yaml --extra-vars "tre_version=$tre_version" /home/ansible/${ostype}/installRemoteWindows.yaml')
            } else {
                sh('docker run -v /home/jenkins/jenkins_home/workspace/Talend-Remote-Engine-Windows:/home/ansible --env AZURE_SUBSCRIPTION_ID=$AZURE_SUBSCRIPTION_ID --env AZURE_CLIENT_ID=$AZURE_CLIENT_ID --env AZURE_SECRET=$AZURE_CLIENT_SECRET --env AZURE_TENANT=$AZURE_TENANT_ID --rm 172.22.6.131:8083/devops/ansible:1.0 ansible-playbook -vvv -i /home/ansible/${ostype}/azure_rm.yaml -u azureuser --private-key /home/ansible/id_rsa --extra-vars "tre_version=$tre_version ansible_host_key_checking=false" /home/ansible/${ostype}/installRemoteUbuntu.yaml')
            }
        }
    }
}
