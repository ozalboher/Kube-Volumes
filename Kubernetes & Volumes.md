# Kubernetes & Volumes:
## This tutorial will cover the importance of volumes in Kubernetes and how to use them effectively.
## What are Volumes in Kubernetes?
- Volumes are attached to PODS.
- By default they survive a CONTAINER restart but not a POD restart/removal.
- In a similar way to docker, the docker volume is stored in my host machine. And a kubernetes volume is stored inside the worker node - that machine in the cloud - if removed basically is like me removing the local machine in docker - which is why the volume will be removed.

# Types of Volumes:
## 1. EmptyDir (POD independent volume)
- It will persist data if the CONTAINER crashes or is restarted.
- It is deleted when the POD is removed. So when POD is then recreated - a new emptyDir (empty directory) is created, ready to store data.
- It is stored in the worker node's filesystem (in a default address) - but tied to the lifecycle of the POD.
- we set it up in: 
1. the deployment.yaml file -> template(pod) -> spec -> volumes. (defines the available volumes to be usable by a pod/s)
2. template(pod) -> spec -> containers -> volumeMounts. (define where which volume will be mounted in the container)
```yaml
volumes:
  - name: my-emptydir-volume
    emptyDir: {}
```
```yaml
containers:
  - name: my-container
    image: image-name
    volumeMounts:
      - name: my-emptydir-volume
        mountPath: /app/story
```
-> You can check this by crashing the app on purpose (/error) and check to see if the data is still there. kubectl get pods will show 1 restarts, which means the container crashed and was restarted not the pod itself, but if you delete the pod with kubectl delete pod <pod_name> and then recreate it (will be created automatically), the data will be lost.

## 2. HostPath (Worker Node independent volume)
-  It will persist data if the CONTAINER and POD crashes or is restarted.
- It is deleted when the WORKER NODE is removed/deleted.
- The volume is now tied to the worker node, not the POD like the emptyDir, but on situations where I might want to share the data between multiple worker nodes, It will not be possible.(in minikube there is only one simulated worker node, that acts as a master node too, so it is not a problem but for production maybe it is not a good idea to use hostPath)
- It is stored in the worker node's filesystem in a specific path (which you define) - and like bind mount will be tied the POD.
- You will specify the path to the folder where the data will be stored, and that could already be an existing folder(with some data in it) or a new one. If a path points to a folder which is not yet created, you need to specify that under the hostPath section -> type: DirectoryOrCreate.(or just Directory if the folder already exists)
```yaml
volumes:
  - name: my-hostpath-volume
    hostPath: /data
    type: DirectoryOrCreate
```
```yaml
containers:
  - name: my-container
    image: image-name
    volumeMounts:
      - name: my-hostpath-volume
        mountPath: /app/story
```
# 3. PersistentVolume (PV) and PersistentVolumeClaim (PVC)
- Presistent Volumes are a place to have storage that is DETACHED from the POD and WORKER NODE. This is a step further than the emptyDir and hostPath volumes, we covered so far.
- So in this method, we will have a whole independent yaml file to define the persistent volume. we can name it "host-pv.yaml" or something similar. open the file to see further explanations. And another file to define the persistent volume claim, which is like a request for storage. we can name it "host-pvc.yaml".
- Note that for this tutorial since we are using minikube, we will use the hostPath type of persistent volume, but in production, we will use the "CSI" volume type with matching cloud provider drivers to connect to cloud storage services.
- In the accessMode option, you can list all the access modes you want to allow for the volume. For example, if you want to allow ReadWriteMany, you can add it to the list. You can define multiple cases -> and when you claim the volume, you can specify which access mode you want to use. 
- In our learnig case, since minikube is running on a single node, we will use the ReadWriteOnce access mode. which means that the volume can be mounted as read-write by a single node.
- Not all volume types support all access modes so check if it is supported by the volume type you are using.