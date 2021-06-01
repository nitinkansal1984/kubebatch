Below are example of Security Context in the Container.

.spec.containers[].securityContext.privileged — The field tells kubelet to run the container in the privileged mode. Processes in privileged containers are essentially identical to root processes on the host. The default value is false.

.spec.containers[].securityContext.readOnlyRootFilesystem — Defines whether a container has a read-only root filesystem. The default value is false.

.spec.containers[]securityContext.runAsUser — The same as in the PodSecurityContext


If you want a fine-grained control over process privileges, you can use Linux capabilities.

Add capability for NET_ADMIN

securityContext:               # ip route add 10.0.1.0/24 via 10.1.0.68 and date -s '19 APR 2012 11:14:00'
   capabilities:
     add: ["NET_ADMIN", "SYS_TIME"]

