    - Verify fluxctl is installed in your cluster
     fluxctl version
     
    - check if kubernetes cluster nodes are visible
     kubectl get nodes
     
    - verify that the namespaces "lasample" does not already exist as we are creating this in the content-gitops repo using manifest files
     kubectl get namespaces
  
    - Install flux in your kubernetes cluster
     fluxctl install \
     --git-user=${GHUSER} \
     --git-email=${GHUSER}@users.noreply.github.com \
     --git-url=git@github.com:${GHUSER}/content-gitops \
     --git-path=namespaces,workloads \
     --namespace=flux | kubectl apply -f -
       
       **Flux v1 is deprecated, please upgrade to v2 as soon as possible!**

      New users of Flux can Get Started here:
      https://fluxcd.io/docs/get-started/

      Existing users can upgrade using the Migration Guide:
      https://fluxcd.io/docs/migration/

      secret/flux-git-deploy created
      deployment.apps/memcached created
      service/memcached created
      serviceaccount/flux created
      clusterrole.rbac.authorization.k8s.io/flux created
      clusterrolebinding.rbac.authorization.k8s.io/flux created
      deployment.apps/flux created

      Note: All above objects are created using https://github.com/usunnapu/content-gitops/blob/master/docs/Flux_Install_Yaml.md
    
    - verify that all pods/resources under flux namespace are up/running
     kubectl get all -n flux
     
     cloud_user@ip-10-0-1-101:~$ kubectl get all -n flux
      NAME                             READY   STATUS    RESTARTS   AGE
      pod/flux-5fb98d7c86-fjb8g        1/1     Running   0          14m
      pod/memcached-6bd86b7b9d-9wpwg   1/1     Running   0          14m

      NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
      service/memcached   ClusterIP   10.105.71.60   <none>        11211/TCP   14m

      NAME                        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
      deployment.apps/flux        1         1         1            1           14m
      deployment.apps/memcached   1         1         1            1           14m

      NAME                                   DESIRED   CURRENT   READY   AGE
      replicaset.apps/flux-5fb98d7c86        1         1         1       14m
      replicaset.apps/memcached-6bd86b7b9d   1         1         1       14m

    - Invoke the "fluxctl identity" and sync the git repository so that manifest files under directories namespaces, workloads are applied
      fluxctl identity --k8s-fwd-ns flux
      fluxctl sync --k8s-fwd-ns flux
   
    - now verify that the namespaces defined under "namespaces" directory are created
    
      cloud_user@ip-10-0-1-101:~$ kubectl get namespaces
      NAME          STATUS   AGE
      default       Active   12m
      flux          Active   5m5s
      kube-public   Active   12m
      kube-system   Active   12m
      lasample      Active   32s
  
    - verify that the manifest files under "workloads" directory is applied and is up/running in namespace "lasample"
      cloud_user@ip-10-0-1-101:~$ kubectl get all -n lasample
      NAME                                    READY   STATUS         RESTARTS   AGE
      pod/nginx-deployment-5c689d88bb-59ztf   0/1     ErrImagePull   0          16m
      pod/nginx-deployment-5c689d88bb-hfqr5   1/1     Running        0          16m

      NAME                               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
      deployment.apps/nginx-deployment   2         2         2            1           16m

      NAME                                          DESIRED   CURRENT   READY   AGE
      replicaset.apps/nginx-deployment-5c689d88bb   2         2         1       16m
      cloud_user@ip-10-0-1-101:~$

