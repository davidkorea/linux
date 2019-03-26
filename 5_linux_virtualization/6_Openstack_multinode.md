


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

FAILED - RETRYING: Waiting for master mariadb (2 retries left).
FAILED - RETRYING: Waiting for master mariadb (1 retries left).
fatal: [server162]: FAILED! => {"attempts": 10, "changed": false, "module_stderr": "Shared connection to server162 closed.\r\n", "module_stdout": "Traceback (most recent call last):\r\n  File \"/root/.ansible/tmp/ansible-tmp-1553493174.26-227571964728856/AnsiballZ_wait_for.py\", line 113, in <module>\r\n    _ansiballz_main()\r\n  File \"/root/.ansible/tmp/ansible-tmp-1553493174.26-227571964728856/AnsiballZ_wait_for.py\", line 105, in _ansiballz_main\r\n    invoke_module(zipped_mod, temp_path, ANSIBALLZ_PARAMS)\r\n  File \"/root/.ansible/tmp/ansible-tmp-1553493174.26-227571964728856/AnsiballZ_wait_for.py\", line 48, in invoke_module\r\n    imp.load_module('__main__', mod, module, MOD_DESC)\r\n  File \"/tmp/ansible_wait_for_payload_xQGt1V/__main__.py\", line 629, in <module>\r\n  File \"/tmp/ansible_wait_for_payload_xQGt1V/__main__.py\", line 558, in main\r\nsocket.error: [Errno 104] Connection reset by peer\r\n", "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error", "rc": 1}

NO MORE HOSTS LEFT *************************************************************
	to retry, use: --limit @/usr/share/kolla-ansible/ansible/site.retry

PLAY RECAP *********************************************************************
client163                  : ok=24   changed=0    unreachable=0    failed=0   
client164                  : ok=24   changed=0    unreachable=0    failed=0   
server162                  : ok=48   changed=2    unreachable=0    failed=1   
```
3. kolla-ansible -i ./multinode bootstrap-servers

```
PLAY RECAP *********************************************************************
client163                  : ok=37   changed=6    unreachable=0    failed=0   
client164                  : ok=37   changed=6    unreachable=0    failed=0   
server162                  : ok=38   changed=18   unreachable=0    failed=0   
```
