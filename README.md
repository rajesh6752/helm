Kubernetes Helm

Helm:-
1. It is a package manager just like yum or apt-get.
2. Helm has two component:-
a) helm:- it is a binary file and installed on workstation(any machine).
b) tiller:- It is a server side component, it is just a pod and it responsible to perform task on kubernetes cluster.

3. In helm, the packages or application are called chart. ex. MySQL, redis, Jenkins, WordPress etc.
4. It is for repetitive task. if you want to deploy common application multiple times then you can use helm to deploy a chart for same.
5. we can customized or over right setting of charts either command-line or using file.
6. When you install helm then it check .kube/config file to understand which cluster need to connect and deploy the tiller.
7. To ensure security, we need to create service account, cluster role binding for tiller.
8. The configuration of helm will be store under directory ~/.helm. Initially this directory will not be present, It will be created when helm is initialized.
Chart:- It contains package or application information such as chart version or application version. below is 3 components are mandatory for the charts.
example of charts:-
apiVersion: v1
name: my-nginx
version: 0.1.0
Templates:- templates directory is contains all the yaml files that gets deployed.
Some helm commands:-
helm helphelm install package-name ( — values) ( — name).
helm fetch package-name => to fetch chart locally to your machine.
helm list => it list the installed package or application.
helm status => it show throw the status of the application  deployment.

helm search => search for the chart.
helm repo update => it use to update the repo.
helm upgrade => to upgrade the chart.
helm rollback => to rollback to any previous version of chart.
helm delete ( — pergue) => delete the deployment. if you want to delete completely with revision.
helm reset ( — force) => it will delete the completely the tiller in your cluster.
Installing helm 2.x:-
wget https://get.helm.sh/helm-v2.16.10-linux-amd64.tar.gz
tar xzvf helm-v2.16.10-linux-amd64.tar.gz
cd linux-amd64/
sudo mv helm /usr/local/bin/
which helm
helm version — short
Installing Tiller:-
#Create clusterRoleBinding
kubectl create clusterrolebinding tiller — clusterrole cluster-admin — serviceaccount=kube-system:tiller
kubectl get clusterrolebinding tiller
#Initilised Helm
helm init — service-account tiller
#Now tiller pod will be created and the help conf directory also has been created.
kubectl get po -n kube-system
ls -l ~/.helm/
Installation of helm 3.x:-
# curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
# chmod 700 get_helm.sh
# ./get_helm.sh
Note:- you don’t need to install tiller for helm 3.x.
Some helm example:-
#Check the helm default repository
helm repo list
#Check installed application
helm list
#Check any application
helm search jenkins
#Check what's inside application
helm inspect stable/jenkins
#Download the chart
helm fetch stable/jenkins
ls
kubectl get deploy,replicaset,pod,serviceaccount,clusterrolebinding -n kube-system | grep tiller
#Delete helm completely
helm reset — remove-helm-home
Create a custom helm chart by own:-
1. first create a directory with same name that application you want to deploy.
# mkdir my-nginx
2. create Chart.yaml for the application metadata
# cd mkdir my-nginx
# vim Chart.yaml
3. create directory with name ‘templates’ to store your actual yaml files.
# mkdir templates
4. create all yaml files inside templates directory
# kubectl create deploy my-ngunx — image nginx — dry-run -o yaml > deployment.yml
5. install the chart
#helm install my-nginx .
If you want to update your existing chart then :-
add or update the yaml files
# kubectl expose deploy my-ngunx — port 80 — dry-run -o yaml > templates/service.yml
2. update the chart version
# vim Chart.yaml
3. upgrade the deployed chart
# helm upgrade my-nginx .
If you want to rollback deployed helm chart to any particular revision then:-
if you want to rollback it to revision 1 then.
# helm rollback my-nginx 1
Delete the chart:-
# helm delete my-nginx
How to Parameterized the chart:-
Add values.yaml file inside your base directory
# vim values.yml
replicaCount: 1
2. Update the templates/deployment.yml files
# vim templates/deployment.yml
replicas: {{ .Values.replicaCount }}
3. Update the chartVersion in Chart.yaml file
vim Chart.yaml
4. Install the chart
# helm install my-nginx . — set replicaCount=2
 or
# helm inspect values . > /tmp/my-nginx.values
now update the replicaCount in the files
# vim /tmp/my-nginx.values
deploy the helm
# helm install my-nginx . — values > /tmp/my-nginx.values
Now Parameterized the service as well
update the values.yaml files as below
# vim values.yaml
replicaCount: 1
service:
 type: NodePort
2. update the templates/service.yml file
# vim templates/service.yml
spec:
 type: {{ .Values.service.type }}
3. update the Chart version as well
# vim Chart.yml
4. upgrade the deployed helm chart
# helm upgrade my-nginx . — set service.type=NodePort
# helm upgrade my-nginx .  — set service.type=LoadBalancer  — set replicaCount=2
Alternatively you can create helm chart by using helm create <appname>. then you will get some set of files and directory.
How to migrate helm v2 to v3:-
1. First we need to migrate helm package and configuration
2. And then we need to migrate installed charts
helm install stable/mysql — name mysql
helm list
#Check any installed plugin
helm plugin list
helm3 plugin list
#Install helm-2to3 plugin to migrate helm v2 to v3
helm3 plugin install https://github.com/helm/helm-2to3
helm3 plugin list
## plugin 2to3 options
helm3 2to3 help
# move the helm2 config to helm3
helm3 2to3 move config — dry-run
helm3 2to3 move config
#Now you can get default repository of helm3
helm3 repo list
# Now we have to migrate application from v2 to v3 one by one
helm3 2to3 convert mysql — dry-run
helm3 2to3 convert mysql
# Check the application
helm3 list
kubectl get all
# Delete the helm2
helm3 2to3 cleanup
# Re-Install again
helm3 repo update
helm3 search repo mysql
helm3 install mysql stable/mysql
helm3 list
Diff b/w helm v2 and v3:-
1. The main diff b/w helm v2 and v3 is that, in v2 we have to install both client side and server side component(helm and tiller). However in helm v3 we do not need to install server side component(tiller).

2. Since we are not using tiller in v3 then we we do not need to create serviceAccount and clusterRoleBinding but in v2 we was using tiller component so for that we had to create serviceAccount and clusterRoleBinding.

3. In v3, helm is using .kube/config file so whatever access you have given to that kube/config file, So that the access it would have to helm.
4. In helm v2 we got a default repository but in v3 we do not get any default repository, we need to add it.
