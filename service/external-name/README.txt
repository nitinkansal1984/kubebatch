Step 1) Launch EKS cluster, and client machine.

Install eksctl

 # curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
 # sudo mv /tmp/eksctl /usr/local/bin
 # eksctl version

Download kubeconfig

 # aws configure
 # aws eks update-kubeconfig --name example
 # aws eks update-kubeconfig --name eks

Install kubectl

 # curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
 # sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
 # kubectl get nodes


Step 2) Launch RDS instance mysql based with no public subnet

Step 3) Create mysql service

# Replace Host Name of Azure MySQL Database and Username and Password
kubectl run -it --rm --image=mysql:5.7.22 --restart=Never mysql-client --
mysql -h akswebappdb.mysql.database.azure.com -u dbadmin@akswebappdb
-pRedhat1449

mysql> show schemas;
mysql> create database webappdb;
mysql> show schemas;
mysql> exit

Step 4) Create webservice and external service to access webservice by
applying rest of the manifest files
