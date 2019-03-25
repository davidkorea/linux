


1. kolla-ansible -i /etc/kolla/multinode prechecks
```
PLAY RECAP *********************************************************************
client163                  : ok=3    changed=0    unreachable=0    failed=1   
client164                  : ok=7    changed=0    unreachable=0    failed=1   
server162                  : ok=67   changed=0    unreachable=0    failed=0  
```
2. [root@server162 kolla]# kolla-ansible -i ./multinode deploy

```
TASK [mariadb : include_tasks] *************************************************
included: /usr/share/kolla-ansible/ansible/roles/mariadb/tasks/bootstrap_cluster.yml for server162

TASK [mariadb : Running MariaDB bootstrap container] ***************************
fatal: [server162]: FAILED! => {"changed": true, "msg": "Container exited with non-zero return code 1"}

RUNNING HANDLER [mariadb : restart slave mariadb] ******************************

RUNNING HANDLER [mariadb : restart master mariadb] *****************************
	to retry, use: --limit @/usr/share/kolla-ansible/ansible/site.retry

PLAY RECAP *********************************************************************
client163                  : ok=29   changed=20   unreachable=0    failed=0   
client164                  : ok=29   changed=20   unreachable=0    failed=0   
server162                  : ok=53   changed=30   unreachable=0    failed=1   
```
reboot and try again
```
RUNNING HANDLER [mariadb : Waiting for master mariadb] *************************
FAILED - RETRYING: Waiting for master mariadb (10 retries left).
FAILED - RETRYING: Waiting for master mariadb (9 retries left).
FAILED - RETRYING: Waiting for master mariadb (8 retries left).
```
