# summittalk2017-omnichannel
# References: https://github.com/DuncanDoyle/alexa-coolstore-microservice
Deploy Mortgage Demo: https://github.com/redhatdemocentral/rhcs-mortgage-demo
oc login
oc new-project rhcs-mortgage-demo 
oc new-build "jbossdemocentral/developer" --name=rhcs-mortgage-demo --binary=true
oc import-image developer
oc start-build rhcs-mortgage-demo --from-dir=. --follow=true --wait=true
oc new-app rhcs-mortgage-demo
oc expose service rhcs-mortgage-demo --hostname=??
http://rhcs-mortgage-demo.10.1.2.2.xip.io/business-central 
[ u:erics / p:jbossbpm1! ] 
Directory /opt/jboss/bpms/jboss-eap-6.4/standalone/data/content is not writable
	at org.jboss.as.repository.ContentRepository$Factory$ContentRepositoryImpl.<init>(ContentRepository.java:177)
  
 
 oc new-app --template=processserver63-mysql-persistent-s2i -p APPLICATION_NAME=policyquote,SOURCE_REPOSITORY_URL=https://github.com/anurag-saran/bxms-xpaas-policyquote-process.git,CONTEXT_DIR=policyquote-process,KIE_SERVER_PASSWORD=kieserver1!,IMAGE_STREAM_NAMESPACE=summittalk2017-dev,KIE_CONTAINER_DEPLOYMENT="policyquote-process=com.redhat.gpte.xpaas.process-server:policyquote-process:1.0-SNAPSHOT",KIE_CONTAINER_REDIRECT_ENABLED=false
 ,MAVEN_MIRROR_URL=$nexus_url/content/groups/public/
 
# Decision Server
cd /Users/anuragsaran/Documents/MW/summit2017/bxms-advanced-infrastructure-lab/xpaas/decision-server
oc project brms-server-policyquote
oc create -f decisionserver-63-is.yaml
oc create -f decisionserver-basic-s2i.yaml
export application_name=policyquote-app
export source_repo=https://github.com/anurag-saran/bxms-xpaas-policyquote.git
export nexus_url=http://nexus-nexus.apps.anuragsdemo.com/
export kieserver_password=kieserver1!
export is_namespace=brms-server-policyquote
export kie_container_deployment="policyquote=com.redhat.gpte.xpaas:policyquote:1.0-SNAPSHOT"

oc new-app --template=decisionserver63-basic-s2i -p KIE_SERVER_PASSWORD=$kieserver_password,APPLICATION_NAME=$application_name,SOURCE_REPOSITORY_URL=$source_repo,IMAGE_STREAM_NAMESPACE=$is_namespace,KIE_CONTAINER_DEPLOYMENT=$kie_container_deployment,KIE_CONTAINER_REDIRECT_ENABLED=false,MAVEN_MIRROR_URL=$nexus_url/content/groups/public/

export policyquote_app=<URL of the policyquote app route>
export policyquote_app=policyquote-app-brms-server-policyquote.apps.anuragsdemo.com

export kieserver_password=kieserver1!
curl -X GET -H "Accept: application/json" --user kieserver:$kieserver_password "$policyquote_app/kie-server/services/rest/server"
curl -X GET -H "Accept: application/json" --user kieserver:$kieserver_password "$policyquote_app/kie-server/services/rest/server/containers"

curl -s -X POST -H "Content-Type: application/json" -H "Accept: application/json" --user kieserver:$kieserver_password -d @policyquote-payload.json "$policyquote_app/kie-server/services/rest/server/containers/instances/policyquote"


# Process Server

cd /Users/anuragsaran/Documents/MW/summit2017/bxms-advanced-infrastructure-lab/xpaas/process-server
oc project bpms-server-policyquote
oc create -f processserver-mysql-persistent-s2i.yaml
oc create -f processserver-63-is.yaml
export application_name=policyquote
export source_repo=https://github.com/anurag-saran/bxms-xpaas-policyquote-process.git
export context_dir=policyquote-process
export nexus_url=http://nexus-nexus.apps.anuragsdemo.com/
export kieserver_password=kieserver1!
export is_namespace=bpms-server-policyquote
export kie_container_deployment="policyquote-process=com.redhat.gpte.xpaas.process-server:policyquote-process:1.0-SNAPSHOT"

oc new-app --template=processserver63-mysql-persistent-s2i -p APPLICATION_NAME=$application_name,SOURCE_REPOSITORY_URL=$source_repo,CONTEXT_DIR=$context_dir,KIE_SERVER_PASSWORD=$kieserver_password,IMAGE_STREAM_NAMESPACE=$is_namespace,KIE_CONTAINER_DEPLOYMENT=$kie_container_deployment,KIE_CONTAINER_REDIRECT_ENABLED=false,MAVEN_MIRROR_URL=$nexus_url/content/groups/public/



export policyquote_app=<URL of the policyquote app route>
cd /Users/anuragsaran/Documents/MW/summit2017/bxms-advanced-infrastructure-lab/xpaas/process-server
export kieserver_password=kieserver1!
export policyquote_app=policyquote-bpms-server-policyquote.apps.anuragsdemo.com
export kieserver_password=kieserver1!
curl -X GET -H "Accept: application/json" --user kieserver:$kieserver_password "$policyquote_app/kie-server/services/rest/server"
curl -X GET -H "Accept: application/json" --user kieserver:$kieserver_password "$policyquote_app/kie-server/services/rest/server/containers"

Start a process
curl -X POST -H "Accept: application/json" -H "Content-Type: application/json" --user kieserver:$kieserver_password -d @policyquote-start-process-payload.json "$policyquote_app/kie-server/services/rest/server/containers/policyquote-process/processes/policyquote.PolicyQuoteProcess/instances"

Check that the process instance is running:
curl -X GET -H "Accept: application/json" --user kieserver:$kieserver_password "$policyquote_app/kie-server/services/rest/server/queries/containers/policyquote-process/process/instances"

Query for the tasks that have user1 as potential owner:
curl -X GET -H "Accept: application/json" --user user1:user "$policyquote_app/kie-server/services/rest/server/queries/tasks/instances/pot-owners"

As user1, claim and start the task:
curl -X PUT -H "Accept: application/json" --user user1:user "$policyquote_app/kie-server/services/rest/server/containers/policyquote-process/tasks/1/states/claimed"
$ curl -X PUT -H "Accept: application/json" --user user1:user "$policyquote_app/kie-server/services/rest/server/containers/policyquote-process/tasks/1/states/started"

Again as user1, complete the task specifying the policy price as payload of this call, using the task_price task output variable:
$ curl -X PUT -H "Accept: application/json" --user user1:user -d '{ "task_price" : 300 }' "$policyquote_app/kie-server/services/rest/server/containers/policyquote-process/tasks/1/states/completed"

Obtain the tasks definitions in the process as the kieserver user, including the input and output data associations:
$ curl -X GET -H "Accept: application/json" --user kieserver:$kieserver_password "$policyquote_app/kie-server/services/rest/server/containers/policyquote-process/processes/definitions/policyquote.PolicyQuoteProcess/tasks/users"
**********Kavita's Project******************

cd /Users/anuragsaran/Documents/MW/summit2017/bxms-advanced-infrastructure-lab/xpaas/process-server
oc new-project bpms-server-kavitha
oc create -f processserver-mysql-persistent-s2i.yaml
oc create -f processserver-63-is.yaml
export application_name=summitexpenseprj
export source_repo=https://github.com/kasriniv/stunning-fiesta.git
export context_dir=summitExpensePrj
export nexus_url=http://nexus-nexus.apps.anuragsdemo.com/
export kieserver_password=kieserver1!
export is_namespace=bpms-server-kavitha
export kie_container_deployment="summitExpensePrj=Summit:summitExpensePrj:1.5"

oc new-app --template=processserver63-mysql-persistent-s2i -p APPLICATION_NAME=$application_name,SOURCE_REPOSITORY_URL=$source_repo,CONTEXT_DIR=$context_dir,KIE_SERVER_PASSWORD=$kieserver_password,IMAGE_STREAM_NAMESPACE=$is_namespace,KIE_CONTAINER_DEPLOYMENT=$kie_container_deployment,KIE_CONTAINER_REDIRECT_ENABLED=false,MAVEN_MIRROR_URL=$nexus_url/content/groups/public/


cd /Users/anuragsaran/Documents/MW/summit2017/bxms-advanced-infrastructure-lab/xpaas/process-server
export kieserver_password=kieserver1!
export policyquote_app=summitexpenseprj-bpms-server-kavitha.apps.anuragsdemo.com
export kieserver_password=kieserver1!
curl -X GET -H "Accept: application/json" --user kieserver:$kieserver_password "$policyquote_app/kie-server/services/rest/server"
curl -X GET -H "Accept: application/json" --user kieserver:$kieserver_password "$policyquote_app/kie-server/services/rest/server/containers"

*********** Mortgage ************
cd /Users/anuragsaran/Documents/MW/summit2017/bxms-advanced-infrastructure-lab/xpaas/process-server
oc new-project bpms-server-mortgage-dev
oc create -f processserver-mysql-persistent-s2i.yaml
oc create -f processserver-63-is.yaml
export application_name=mortgage
export source_repo=https://github.com/anurag-saran/rhcs-mortgage-demo-hook.git
export context_dir=MortgageApplication
export nexus_url=http://nexus-nexus.apps.anuragsdemo.com/
export kieserver_password=kieserver1!
export is_namespace=bpms-server-mortgage-dev
export kie_container_deployment="mortgage=com.redhat.bpms.examples:mortgage:5"

oc new-app --template=processserver63-mysql-persistent-s2i -p APPLICATION_NAME=$application_name,SOURCE_REPOSITORY_URL=$source_repo,CONTEXT_DIR=$context_dir,KIE_SERVER_PASSWORD=$kieserver_password,IMAGE_STREAM_NAMESPACE=$is_namespace,KIE_CONTAINER_DEPLOYMENT=$kie_container_deployment,KIE_CONTAINER_REDIRECT_ENABLED=false,MAVEN_MIRROR_URL=$nexus_url/content/groups/public/


cd /Users/anuragsaran/Documents/MW/summit2017/bxms-advanced-infrastructure-lab/xpaas/process-server
export kieserver_password=kieserver1!
export policyquote_app=mortgage-bpms-server-mortgage-dev.apps.anuragsdemo.com
export kieserver_password=kieserver1!
curl -X GET -H "Accept: application/json" --user kieserver:$kieserver_password "$policyquote_app/kie-server/services/rest/server"
curl -X GET -H "Accept: application/json" --user kieserver:$kieserver_password "$policyquote_app/kie-server/services/rest/server/containers"


**********Git Hook**********
https://access.redhat.com/documentation/en-us/red_hat_jboss_bpm_suite/6.4/html/administration_and_configuration_guide/chap_repository_hooks#configuring_git_hooks

sh-4.2$ cat /opt/jboss/bpms/jboss-eap-6.4/bin/.niogit/Mortgages.git/config
[core]
	symlinks = false
	repositoryformatversion = 0
	filemode = true
	bare = true
	logallrefupdates = false
	precomposeunicode = true
[remote "origin"]
	url = http://anurag-saran:Password@github.com/anurag-saran/rhcs-mortgage-demo-hook.git
	fetch = +refs/heads/*:refs/heads/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
[credential]
	helper = store
sh-4.2$ 

sh-4.2$ cat /opt/jboss/bpms/jboss-eap-6.4/bin/.niogit/Mortgages.git/hooks/post-commit             
#!/bin/sh
git push origin master
sh-4.2$ 

****** Another Approach *********
system property for external git access: 
org.uberfire.nio.git.ssh.daemon.host=192.168.x.x






