# KVM in Apigee

In Apigee, KVM stands for Key-Value Map. KVM is a feature provided by Apigee that allows you to store and retrieve key-value pairs within the Apigee Edge platform. It acts as a lightweight data store that can be used by API proxies and other components of your API ecosystem.

## Create or Update KVM in Apigee using Jenkins

This repository contains Jenkinsfile which has script for creating/updating kvm in Apigee.

For using this template we have to update our Apigee organisation name and credentials in jenkinsfile for parameter:

`def orgList = ['APIGEE_ORGANISATION_NAME']`

`credentialsId: '<gcp_service_account>'`


Create a Jenkins pipeline using this repository and build that pipeline by passing parameters to it. You can select multiple envs at the same time in parameter. 


![jenkins_pipeline](https://i.postimg.cc/kMpLhVnF/Screenshot-from-2023-07-15-18-32-03.png)


Build the pipeline and it will create/update KVM in Apigee.

To verify this move to:

Apigee > Admin > Environments > Key Value Maps
