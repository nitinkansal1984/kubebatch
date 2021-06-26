kubectl run frontend --image nginx
kubectl run backend --image nginx
kubectl expose pod frontend --port 80
kubectl expose pod backend --port 80

check connectivity:

kubectl exec  frontend -- curl backend

kubectl exec  backend -- curl frontend

Step1)
Create Network Policy default_deny.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  egress:
  - to:
    ports:
     - port: 53
       protocol: TCP
     - port: 53
       protocol: UDP


kubectl apply -f default_deny.yaml

Test the connectivity again.

kubectl exec  frontend -- curl backend
kubectl exec  backend -- curl frontend


Step2)
Allow frontend to backend based on podSelector:

frontend.yaml

Go to documentation and paste it here and adjust it

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: frontend
  policyTypes:
  - Egress
  egress:
  - to:
     - podSelector:
        matchLabels:
          run: backend

kubectl apply -f frontend.yaml

Verify it again:

Create another policy to allow ingress to backend:

Copy and paste the last policy and change it.

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
     - podSelector:
        matchLabels:
          run: frontend


Verify again from frontend to backend.

kubectl exec frontend -- curl backend

This is one way only. We didn't allow anything from backend to frontend:

so if you run below command, it will fail.

# kubectl exec backend -- curl <frontend>


Step 3) Namespace Selector:

kubectl create ns cassandra


add some label to it, which we will use in network policy.

kubectl edit ns cassandra

labels:
   ns: cassandra

kubectl run cassandra --image nginx -n cassandra

Now that we have default deny already, if you try connect backend to cassandra pod, it will fail

kubectl exec backend -- curl <cassandra>

change backend policy and add:

kubectl apply -f 4.from_backend_to_cassandra_namespace.yaml 

kubectl exec backend -- curl cassandra

Now Add default deny policy in cassandra namespace as good practice.

kubectl apply -f 5.default_deny_cassandra_namespace.yaml

Verify the connectivity again.

kubectl exec backend -- curl cassandra

Now add allow from default namespace to cassandra namespace. 

kubectl apply -f .\6.allow_cassandra_from_default_ns.yaml 

NOTE: Make sure we have label added in default namespace






