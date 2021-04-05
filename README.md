# clamav_monitoring
ClamAV monitoring for OpenShift clusters

In this demo, we create a custom Docker image that mounts its host server's file system and then scans that folder at regular intervals using OpenSource ClamAV. https://www.clamav.net/

Steps:

1. Clone the GitHub repository:
```
git clone https://github.com/IBM-ICP4D/clamav-monitoring
```

2. Navigate to the working directory:
```
cd clamav_monitoring
```

3. Customize the antivirus scan options in the script scan.sh and for the directories to be scanned as needed. For more information, see the official configuration documentation at https://www.clamav.net/documents/configuration . 

Some examples are 

3.1. Scan all folders in /host-fs excluding certain directories


```
	clamscan \
    --verbose \
    --stdout \
    --log=/logs/clamscan.log \
    --recursive \
    --exclude=/host-fs/dev \
    --exclude=/host-fs/sys \
    --exclude=/host-fs/var/lib/docker \
    /host-fs
```

3.2.  Scan multiple directories by mentioning them in an input file


```
	clamscan \
    --verbose \
    --stdout \
    --log=/logs/clamscan.log \
    --recursive \
    --file-list=/folders_to_scan.txt
```


4. Modify start.py to customize the cron schedule.

By default, the system scans for viruses at the same time every hour. Instead of letting the script pick a random minute to execute, you might want to choose a specific scan time or run the scans more or less frequently. You can also customize logging, ignored paths, and performance options. In addition to logging to a file, the image is configured to log to standard output, so that Cloud Monitoring can capture scan logs.

5. Build and push the image to Container Registry by running the following commands in the Dockerfile folder:

```
DOCKER_REPO=<Your docker repo for example default-route-openshift-image-registry.apps.sre-3m15w-nfs-01.cp.fyre.ibm.com>
docker build -t clamav .
docker tag clamav $DOCKER_REPO/clamav-scanner/clamav:1.0.0
docker push $DOCKER_REPO/clamav-scanner/clamav:1.0.0
```

6. Create the namespace and the service account
```
oc create namespace clamav-scanner
oc create serviceaccount clamav-sa -n clamav-scanner
```

7. If you need to run the scanner in a privileged mode as it needs access to the host filesystem. So you need to add privileged scc to the service account
```
oc adm policy add-scc-to-user privileged system:serviceaccount:clamav-scanner:clamav-sa
```

8. Switch to the project created above and run the daemonset by running the commands
```
oc project clamav-scanner
oc create -f clamav-daemonset.yaml
```

9. The scan output can be seen on the worker nodes by looking at the /var/log/clamav/clamscan.log log file or to see only a summary you can grep for the following lines in the log files.

```
egrep 'Starting scan|SCAN SUMMARY|Known viruses|Engine version|Scanned directories|Scanned file|Infected files|Data scanned|Data read|Time:' /var/log/clamav/clamscan.log
```
