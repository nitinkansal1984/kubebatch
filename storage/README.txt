Manual approach:

az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv


$ az disk create \
  --resource-group MC_myResourceGroup_myAKSCluster_eastus \
  --name k8s_volume_1  \
  --size-gb 10 \
  --query id \
  --output tsv

$ cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: mypod-azuredisk
spec:
  containers:
  - image: nginx
    name: mypod
    volumeMounts:
      - name: mystorage
        mountPath: /data
  volumes:
      - name: mystorage
        azureDisk:
          kind: Managed
          diskName: k8s_volume_1
          diskURI: disk_URI
EOF


------------------------------

Kubernetes can dynamically provision Azure Disks using the Azure Kubernetes integration.

For that we need to create storage class which will act as provisioner of storage in cloud provider.  The storage class is a object in kubernetes. You would need to create Persistent Volume Claim directly.
Persistent Volume Claim will in turn create Persistent Volume. The persistent volume claim can then be used in manifest to mount as volume to store data.

You can use pre existing storage class to provision the azure disk dynamically or create your own storage class:


Template for understanding all default and available values:

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-premium-retain-sc
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Retain  # Default is Delete, recommended is retain
volumeBindingMode: WaitForFirstConsumer # Default is Immediate, recommended is WaitForFirstConsumer
allowVolumeExpansion: true  
parameters:
  storageaccounttype: Premium_LRS # or we can use Standard_LRS
  kind: managed # Default is shared (Other two are managed and dedicated)



To create a Standard Storage Class:

cat <<EOF | kubectl create -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: standard
provisioner: kubernetes.io/azure-disk
parameters:
  storageaccounttype: Standard_LRS
  kind: Managed
EOF

To Create a Premium Storage Class:

cat <<EOF | kubectl create -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: premium
provisioner: kubernetes.io/azure-disk
parameters:
  storageaccounttype: Premium_LRS
  kind: Managed
EOF


kubectl get storageclasses

Create an Azure Disk with a Persistent Volume Claim:

After you create a Storage Class, you can use Kubernetes Objects to dynamically provision Azure Disks. This is done using Kubernetes Persistent Volumes Claims.

The following example uses the standard storage class and creates a 5 GiB Azure Disk. Alter these values to fit your use case.

cat <<EOF | kubectl create -f -
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: azure-disk-pvc
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
EOF


Attach the new Azure Disk to a Kubernetes pod 

Now that a Kubernetes Persistent Volume has been created, you can mount this into a Kubernetes Pod. The disk can be consumed by deployment however, the following example just mounts the persistent volume into a standalone pod.

$ cat <<EOF | kubectl create -f -
kind: Pod
apiVersion: v1
metadata:
  name: mypod-dynamic-azuredisk
spec:
  containers:
    - name: mypod
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: storage
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: azure-disk-pvc
EOF

NOW, lets see the same example using other storage i.e. default.


NOTE: Azure Virtual Machine data disk capacity
In Azure, there are limits to the number of data disks that can be attached to each Virtual Machine. This data is shown in Azure Virtual Machine Sizes. Kubernetes is aware of these restrictions, and prevents pods from deploying on Nodes that have reached their maximum Azure Disk Capacity.


-------------
Let's see how azure file share works:






-------------
  
Create Storage Class, PVC, ConfigMap, SQL Deployment, WebApp

manifest file is there in azure_end_to_end.yaml

# Connect to MYSQL Database
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -pdbpassword11

# Verify usermgmt schema got created which we provided in ConfigMap
mysql> show schemas;
mysql> use usermgmt;
mysql> show tables;
mysql> select * from user;
      

-----------

NFS share 

helm install nfs-server stable/nfs-server-provisioner --set persistence.enabled=true,persistence.storageClass=default,persistence.size=10Gi

kubectl get pv
kubectl get pvc
kubectl get pods

kubectl exec -it nfs-server-nfs-server-provisioner-0 -- sh

showmount -e localhost

Create a file in the share.

 