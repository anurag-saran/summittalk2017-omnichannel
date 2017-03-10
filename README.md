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
 
$ export application_name=policyquote
$ export source_repo=$ export source_repo=https://github.com/anurag-saran/bxms-xpaas-policyquote-process.git
$ export context_dir=policyquote-process
$ export nexus_url=http://nexus:8081
$ export kieserver_password=kieserver1!
$ export is_namespace=summittalk2017-dev
$ export kie_container_deployment="policyquote-process=com.redhat.gpte.xpaas.process-server:policyquote-process:1.0-SNAPSHOT"
 oc new-app --template=processserver63-mysql-persistent-s2i -p APPLICATION_NAME=$application_name,SOURCE_REPOSITORY_URL=$source_repo,CONTEXT_DIR=$context_dir,KIE_SERVER_PASSWORD=$kieserver_password,IMAGE_STREAM_NAMESPACE=$is_namespace,KIE_CONTAINER_DEPLOYMENT=$kie_container_deployment,KIE_CONTAINER_REDIRECT_ENABLED=false
 ,MAVEN_MIRROR_URL=$nexus_url/content/groups/public/
