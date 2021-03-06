18% - Configuration 
* Understanding Configmaps
* Understand SecurityContexts
* Define application resource requirements
* Create and consume secrets
* Understand service accounts
### Links you might need to bookmark on your bookmark bar 
* [cm from tasks](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#use-configmap-defined-environment-variables-in-pod-commands)
* [secrets from concepts](https://kubernetes.io/docs/concepts/configuration/secret/#creating-your-own-secrets)
* [secrets from tasks](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#create-a-secret)

#### Create a configmap from literals cmname1=cmvalue1 and cmname2=cmvalue2
```bash
k create cm mycmliteral --from-literal=cmname1=cmvalue1 --from-literal=cmname2=cmvalue2
k describe cm mycmliteral
```
#### Create a configmap from a file names cmfile.txt
```bash
echo -e 'cmfilename=cmfilevalue\nexam=ckad' > cmfile.txt
k create cm myfilecm --from-file=cmfile.txt
k describe cm myfilecm
```
#### create configmap from env file cmenv.txt
```bash
echo -e 'cmenvname=cmenvvalue\nexam=ckad' > cmenv.txt
k create cm cmenv --from-env-file=cmenv.txt 
```
#### create configmap from a folder
```bash
echo -e 'filename1=filevalue1' > /root/cmfolder/file1.txt
echo -e 'filename2=filevalue2' > /root/cmfolder/file2.txt
k create cm cmfolder --from-file=/root/cmfolder
k describe cm cmfolder
```
#### create a configmap from a file with a key alias instead of filename 
```
echo -e 'keyfilename1=keyfilevalue1\nkeyfilename2=keyfilevalue2' > keyfile.txt 
k create cm keyalias --from-file=mykey=keyfile.txt
k describe cm keyalias
```
#### Load configmap value as and env variable named envcmname and print the env variables  
```bash
k create cm myconfig --from-literal=cmname=cmvalue
k run mypod --image=nginx --restart=Never --dry-run -o yaml > pod.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mypod
  name: mypod
spec:
  containers:
  - image: nginx
    name: mypod
    resources: {}
    env:
      - name: envcmname #env varibale name 
        valueFrom:
          configMapKeyRef:
            name: myconfig # configmap name
            key: cmname    # configmap key name
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```bash
k apply -f pod.yaml  
k exec -it mypod -- env
```
#### Create a configMap with multiple name value pairs (name1=value1 and name2=value2) and load them as env variables in a nginx pod
```bash
k create cm cmenv --from-literal=name1=value1 --from-literal=name2=value2
k run mypodenv --image=nginx --restart=Never --dry-run -o yaml > podenv.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mypodenv
  name: mypodenv
spec:
  containers:
  - image: nginx
    name: mypodenv
    resources: {}
    envFrom:
    - configMapRef: # Use configMapRef here for loading name=value pairs as env variables
        name: cmenv
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```bash
k apply -f podenv.yaml
k exec -it mypodenv -- env
```
#### Create a configMap with name values (name1=value1 and name2=value2) and mount configmap as a volume inside a pod to /etc/cmvol
* Two files are created with names name1 and name2 under /etc/cmvol which would have values value1 and value2 respectively 

```bash 
k create cm cmvolume --from-literal=name1=value1 --from-literal=name2=value2
k run mypodcmvol --image=nginx --restart=Never --dry-run -o yaml > podcmvolume.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mypodcmvol
  name: mypodcmvol
spec:
  volumes:
  - name: cmvol
    configMap:
      name: cmvolume
  containers:
  - image: nginx
    name: mypodcmvol
    resources: {}
    volumeMounts:
    - name: cmvol
      mountPath: /etc/cmvol # this path will have files with name's i.e name1 and name2 which have values as content
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}                     
```
```
k apply -f podcmvolume.yaml
k exec -it mypodcmvol -- /bin/bash 
ls -ltra /etc/cmvol
cat name1
cat name2
```
#### Create a configmap from a file named cmfile.txt that has name1=value1 and load it into nginx pod as volume and mount it at /etc/cmfilemount
* After loading the configmap to the volume there will be a file cmfile.txt in /etc/cmfilemount 

```bash 
echo -e 'name1=value1\nname2=value2' > cmfile.txt
k create cm cmfromfile --from-file=cmfile.txt
k run mypodcmfilevol --image=nginx --restart=Never --dry-run -o yaml > podcmfilevolume.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mypodcmfilevol
  name: mypodcmfilevol
spec:
  volumes:
  - name: cmfilevol
    configMap: # configMap only here 
      name: cmfromfile
  containers:
  - image: nginx
    name: mypodcmfilevol
    resources: {}
    volumeMounts:
    - name: cmfilevol
      mountPath: /etc/cmfilemount #cmfile.txt of configmap will be loaded here 
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```
k apply -f pofcmfilevolume.yaml 
k exec -it mypodcmfilevol -- /bin/bash 
ls -ltra /etc/cmfilemount
cat cmfile.txt 
```
#### Create a configmap from literal using name1=value1 and load it into a nginx pod using volume but load the filename as 'namefile' instead of default key name 'name1'
```bash
k create cm myconfig --from-literal=name1=value1 
k run mypod --image=nginx --restart=Never --dry-run -o yaml > pod.yaml 
nano pod.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mypod
  name: mypod
spec:
  volumes:
    - name: cmvol
      configMap:
        name: myconfig
        items: # give items and mention the keyname that need to be replaced
        - key: name1  # key that will be replaced 
          path: namepath # replaced with this name
  containers:
  - image: nginx
    name: mypod
    resources: {}
    volumeMounts:
    - name: cmvol
      mountPath: /etc/cmvol # this path will have namepath instead of name1
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```
```bash
k apply -f pod.yaml 
k exec -it mypod -- /bin/bash 
cat /etc/cmvol/namepath 
```
#### Change the name1 value to value1changed from myconfig cm above and see if the pod reflects the changed value 
```bash
k edit cm myconfig # change value1 to value1changed 
```
```
k annotate po/mypod exam=ckad # as it takes it own time to update the configmap value because of refresh time annotate will make this reload quick
k exec -it mypod -- /bin/bash 
cat /etc/cmvol/namepath # this will give value1changed
```
