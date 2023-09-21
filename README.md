# fio
Quick fio testing

````
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: fio-job-config
data:
  fio.job: |-
    [global]
    ioengine=psync
    direct=1
    buffered=0
    size=10G
    iodepth=1000
    numjobs=100
    group_reporting
    refill_buffers
    rwmixread=80
    norandommap
    randrepeat=0
    percentage_random=0
    bs=512K
    buffer_compress_percentage=50
    rw=read
    [testjob]
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: fio
spec:
  serviceName: fio
  replicas: 3 
  selector:
    matchLabels:
      app: fio
  template:
    metadata:
      labels:
        app: fio
    spec:
      containers:
      - name: fio
        image: quay.io/cloud-bulldozer/fio
        command: ["fio"]
        args: ["/configs/fio.job", "--eta=never", "--filename_format=$jobnum.$filenum", "--directory=/scratch/"]
        volumeMounts:
        - name: fio-config-vol
          mountPath: /configs
        - name: fio-data
          mountPath: /scratch
      volumes:
      - name: fio-config-vol
        configMap:
          name: fio-job-config
  volumeClaimTemplates:
  - metadata:
      name: fio-data
    spec:
      storageClassName: ocs-storagecluster-cephfs
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 100Gi
````
