## Volume 

### emptyDir

An `emptyDir` volume is first created when a Pod is assigned to a Node, and exists as long as that Pod is running on that node. Mounted volumes are specific to pods (even for replica pods) and are not shared across pods.

### Hostpath

A `hostPath` volume mounts a file or directory from the host node’s file-system into Pods. Hostpath is Node specifc and is shared across all pods in that node.

## Persistent Volume

 A `PersistentVolume` (PV) is a piece of storage in the cluster that has been provisioned by an administrator through a `PersistentVolumeCliam` (PVC). 

Claims (PVC) are made against existing PVs, to allocate storage (less than or equal to the available storage of a PV). Claim is then used in pod's .spec.volumes to mount the storage. PV is shared across the pods.

PV can be deleted when it has no active claim, ie. PVCs are deleted. A PVC can be deleted when no pod is using that specific PVC, ie. all pods using that PVC are deleted. 

