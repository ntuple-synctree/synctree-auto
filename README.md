# Caution

<br>

The tasks described in this document should only be performed during the installation phase and should never be performed in the production environment

<br>

# Overview

<br>

This is not a complete silent installation. You must create and register domain, and generate the certificate yourself. This installation will proceed automatically before the corresponding operation.

<br>

After the automatic installation is completed, you can proceed from task "What to Do After You've Completed Installation".

<br>

# Prerequisite

<br>

Start docker
```
sudo systemctl start docker
```

<br>

Check if Active is active (running)
```
systemctl status docker
```
Example output
```
systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2022-09-15 03:19:21 UTC; 2h 2min ago
     Docs: https://docs.docker.com
  Process: 23477 ExecStartPre=/usr/libexec/docker/docker-setup-runtimes.sh (code=exited, status=0/SUCCESS)
  Process: 23467 ExecStartPre=/bin/mkdir -p /run/docker (code=exited, status=0/SUCCESS)
 Main PID: 23492 (dockerd)
    Tasks: 7
   Memory: 21.0M
   CGroup: /system.slice/docker.service
           └─23492 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --default-ulimit nofile=32768:65536

```

<br>

Add users to the docker group for docker execution.
```
sudo usermod -aG docker ec2-user
```

<br>

Logout and reconnect
```
logout
```

<br>

Run 'docker ps' command
```
docker ps
```
Example of output on success
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
Example of output on failure
```
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json": dial unix /var/run/docker.sock: connect: permission denied
```

<br>

# Copy and Modify synctree.yaml

<br>

Copy synctree.yaml to below path
```
cp ~/synctree.yaml ~/install-synctree/group_vars/all/
```

<br>

Move to installtion directory
```
cd ~/install-synctree/
```

<br>

Modify synctree.yaml
```
vi group_vars/all/synctree.yaml
```

<br>

Enter your information in fields studio_username, studio_email and studio_password.
```
vpc_id: vpc-003fdd1e7305a384e
eks_cluster_name: eks-synctree
elasticache_endpoint: synctree-redis.utaiuq.clustercfg.abc2.cache.amazonaws.com
elasticache_port: 6379
eks_min_size: 2
eks_desired_size: 2
eks_max_size: 3
aurora_data_endpoint: synctree-data.cluster-abidowskd7a9.ap-northeast-2.rds.amazonaws.com
aurora_data_username: root
aurora_data_password: Password1!
aurora_data_port: 3306
aurora_log_endpoint: synctree-log.cluster-abidowskd7a9.ap-northeast-2.rds.amazonaws.com
aurora_log_username: root
aurora_log_password: Password1!
aurora_log_port: 3306
alb_version: v2.4.0
cert_manager_version: v1.5.4
studio_username:
studio_email:
studio_password:

```
Example
```
... (omission) ...
studio_username: root
studio_email: root@test.com
studio_password: root1234!
```

<br>

# Run playbook.yaml

<br>

Run the below command. It takes 10 minutes approximately.
```
ansible-playbook playbook.yaml
```

<br>

The installation completion screen is shown below. Check if the failed is 0.
```

... (omission) ...

FAILED - RETRYING: Please wait until aws load balancer is deployed (269 retries left).
FAILED - RETRYING: Please wait until aws load balancer is deployed (268 retries left).
changed: [localhost]

TASK [debug] ***********************************************************************
ok: [localhost] => {
    "result": {
        "attempts": 34,
        "changed": true,
        "cmd": "aws elbv2 describe-load-balancers | yq e '.LoadBalancers[] | select(.LoadBalancerName == \"k8s-synctree-synctree*\") | .State.Code'",
        "delta": "0:00:00.956444",
        "end": "2022-09-14 05:29:11.822154",
        "failed": false,
        "msg": "",
        "rc": 0,
        "start": "2022-09-14 05:29:10.865710",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "active",
        "stdout_lines": [
            "active"
        ]
    }
}

PLAY RECAP ****************************************************************************************
localhost  : ok=114  changed=43   unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
```

<br>

Move to EC2 > Load Balancers and check if state is Active.

![alb_state](https://user-images.githubusercontent.com/103020388/190069926-c830d7f0-8346-43ac-85fa-bd58fc5a2650.png)

<br>

Run the command below to check if STATUS is Running.
```
kubectl get pods -n synctree
```
Example output:
```
NAME                                    READY   STATUS    RESTARTS   AGE
synctree-portal-67b5d5cfd6-fhl8j        1/1     Running   0          41h
synctree-studio-7bf86c4569-xv5b2        1/1     Running   0          41h
synctree-testing-5cb4887677-sbkwt       1/1     Running   0          41h
synctree-tool-6456744d4c-8lnxc          1/1     Running   0          41h
synctree-tool-proxy-5b98d8b74-m6g46     1/1     Running   0          41h
```

<br>

Go to the link below and proceed with the rest of the work.
<br>
https://github.com/ntuple-synctree/synctree#check-load-balancer-creation