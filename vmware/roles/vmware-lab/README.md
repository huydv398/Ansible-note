# Ansible-VMware Part 2
## 1. Liệt kê máy ảo

Liệt kê toàn bộ tên máy ảo, sử dụng kỹ thuật json để list

```
## vmware/roles/vmware-lab/tasks/1.list-vm-vcenter.yml
---
- name: get specific facts from vm
  hosts: localhost
  connection: local
  tasks:
    - name: get facts from vm
      vmware_vm_info:
        hostname: vcenter.vlware.local #IP cua vcenter
        username: administrator@vsphere.local #Username cua vcenter
        password: Suncloud@2022 #Mat khau cua vcenter
        validate_certs: no
      register: vm_info

    - name: show msg
      debug:
        msg: "{{ vm_info.virtual_machines|json_query('[].guest_name') }}"

```

Kết quả:
```
root@ansible:~/Ansible-note# ansible-playbook /root/Ansible-note/vmware/roles/vmware-lab/tasks/1.list-vm-vcenter.yml

PLAY [get specific facts from vm] ******************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************
ok: [localhost]

TASK [get facts from vm] ***************************************************************************************************************************************************************
ok: [localhost]

TASK [show msg] ************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": [
        "LIN-SERVER-01",
        "LAB-U20-Checkmk",
        "labccie",
        "WIN16-TMP",
        "Cloud_Machine_1-mcm486-228450227930",
        "Cloud_Machine_1-mcm459-228418891002",
        "CentOS-7",
        "LAB-U20-NETBOX-CTT2",
        "TEMP-U20",
        "lcm",
        "LIN-SERVER-03",
        "LAB-U20-AnsibleOSS",
        "LAB-JUM-WIN01",
        "LAB-JUM-UBUNTU01",
        "LAB-VCENTER-VLWARE",
        "photon-3-kube-v1.22.9+vmware.1-tkg.1-06852a87cc9526f5368519a709525c68",
        "LAB-C7-SRV01",
        "vra",
        "LAB-U20-SRV01",
        "LAB-BOOTSTRAP01",
        "RedisSentinel01",
        "lab-Netbox",
        "LAB-PROXY",
        "Lab-Ansible-1",
        "LAB-U20-AWX",
        "LAB-UBUNTU-VM01",
        "LAB-UBUNTU-VM02",
        "LAB-JUMP-14-225-16-167",
        "LIN-SERVER-02",
        "HuyDV-Centos7-Client1",
        "Windows-jump",
        "LAB-U20-SRV03",
        "LAB-PRITUNL",
        "LAB-VYOS",
        "LAB-AD01",
        "PRO-JUMP-14-225-16-167",
        "idm",
        "WIN-tmp-test",
        "vrli",
        "LAB-MINIO-01",
        "RedisSentinel03",
        "LAB-MINIO-02",
        "LAB-MINIO-03",
        "LAB-MINIO-04",
        "LAB-U20-HARBOR",
        "vCLS-3c1ae833-0a2d-4c79-8493-f30247e7f86d",
        "RedisSentinel02",
        "LAB-U20-GITLAB",
        "vm-netbox01",
        "vm-netbox02",
        "photon-3-kube-v1.21.11+vmware.1-tkg.1-0262f0ab881e294df81498075207f2b5"
    ]
}

PLAY RECAP *****************************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

## 2. Xóa Máy ảo
playbook để xóa VM, trong play book này có dùng tham số `force: yes` để có thể xóa máy ảo khi đang poweron

Xác định đúng tên máy ảo trước khi xóa, Trong hướng dẫn thực hiện theo 2 kiểu
* Xóa trực tiếp khi sử dụng tên máy ảo.
* Xác định tên máy ảo và lấy ra lấy `uuid` để xóa
```
## vmware/roles/vmware-lab/tasks/2.remove-vm.yml
- name: delete vm
  hosts: localhost
  tasks:

  #Type1:
    # - name: get fact by vm name
    #   vmware_guest:
    #     hostname: vcenter.vlware.local #IP cua vcenter
    #     username: administrator@vsphere.local #Username cua vcenter
    #     password: Suncloud@2022 #Mat khau cua vcenter
    #     validate_certs: no
    #     datacenter: VLWAREDC  #Ten datacenter
    #     folder: /
    #     name: LIN-SERVER-05
    #   register: facts
      
    # - name: Remove a virtual machine by uuid
    #   vmware_guest:
    #     hostname: vcenter.vlware.local #IP cua vcenter
    #     username: administrator@vsphere.local #Username cua vcenter
    #     password: Suncloud@2022 #Mat khau cua vcenter
    #     validate_certs: no
    #     uuid: "{{ facts.instance.hw_product_uuid }}"
    #     force: yes #Luu y: Tham so nay duoc them vao khi VM dang duoc poweron. Nen tat VM truoc khi xoa.
    #     state: absent
  
  ##Type2:      
    - name: Remove a virtual machine by uuid
      vmware_guest:
        hostname: vcenter.vlware.local #IP cua vcenter
        username: administrator@vsphere.local #Username cua vcenter
        password: Suncloud@2022 #Mat khau cua vcenter
        validate_certs: no
        datacenter: VLWAREDC  #Ten datacenter
        name: LIN-SERVER-04
        force: yes #Luu y: Tham so nay duoc them vao khi VM dang duoc poweron. Nen tat VM truoc khi xoa.
        state: absent
```

## 3. Bật tắt VM

### 3.1 Thực hiện bật VM

```
## vmware/roles/vmware-lab/tasks/3.power-vm.yml
- name: poweron vm
  hosts: localhost
  tasks:
    - name: poweron vm
      vmware_guest:
        hostname: vcenter.vlware.local #IP cua vcenter
        username: administrator@vsphere.local #Username cua vcenter
        password: Suncloud@2022 #Mat khau cua vcenter
        datacenter: VLWAREDC
        validate_certs: no
        folder: /
        name: LIN-SERVER-03
        state: poweredon
      register: facts

```
### 3.2 Thực hiện tắt VM
```
## vmware/roles/vmware-lab/tasks/3.power-vm.yml
- name: poweroff vm
  hosts: localhost
  tasks:
    - name: poweroff vm
      vmware_guest:
        hostname: vcenter.vlware.local #IP cua vcenter
        username: administrator@vsphere.local #Username cua vcenter
        password: Suncloud@2022 #Mat khau cua vcenter
        datacenter: VLWAREDC
        validate_certs: no
        folder: /
        name: LIN-SERVER-03
        state: poweredoff
      register: facts

```

## 4. Lấy thông tin máy ảo
Lấy thông tin của máy ảo:
```
### vmware/roles/vmware-lab/tasks/4.info-vm.yml
    - name: get fact by vm name
      vmware_guest:
        hostname: vcenter.vlware.local #IP cua vcenter
        username: administrator@vsphere.local #Username cua vcenter
        password: Suncloud@2022 #Mat khau cua vcenter
        validate_certs: no
        datacenter: VLWAREDC  #Ten datacenter
        folder: /
        name: LIN-SERVER-02
      register: facts
    - name: show msg
      debug:
        msg: "{{ facts }}"
```


```
---
- name: get specific facts from vm
  hosts: localhost
  connection: local
  tasks:
  - name: Gather facts for vm each properties
    vmware_guest_info:
      hostname: vcenter.vlware.local #IP cua vcenter
      username: administrator@vsphere.local #Username cua vcenter
      password: Suncloud@2022 #Mat khau cua vcenter
      validate_certs: no
      datacenter: VLWAREDC  #Ten datacenter
      folder: /
      name: LIN-SERVER-02
      schema: vsphere
      properties:
        - alarmActionsEnabled
        - overallStatus
        - config.name
        - config.annotation
        - config.flags
        - config.managedBy
        - guest.hostName
        - guest.net
        - summary.storage
        - summary.quickStats
        - summary.config
        - summary.runtime
    register: gather_facts_for_vm_each_properties
  
  - name: show msg
    debug: 
      msg: "{{ gather_facts_for_vm_each_properties }}"
```

## 5. Lấy thông tin datacenter

```

```

Kết quả:
```
root@ansible:~/Ansible-note# ansible-playbook /root/Ansible-note/vmware/roles/vmware-lab/tasks/5.get-dc.yml

PLAY [get specific facts from vm] ***********************

TASK [Gathering Facts] **************************
ok: [localhost]

TASK [Gather information about all datacenters] ********
ok: [localhost]

TASK [show msg] ************
ok: [localhost] => {
    "msg": {
        "changed": false,
        "datacenter_info": [
            {
                "config_status": "gray",
                "moid": "datacenter-2040",
                "name": "Datacenterdemo",
                "overall_status": "gray"
            },
            {
                "config_status": "gray",
                "moid": "datacenter-3",
                "name": "VLWAREDC",
                "overall_status": "gray"
            }
        ],
        "failed": false
    }
}

PLAY RECAP **************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
# 2. Sử dụng Kiểu URI
## 2.1 Playbook tương tác với VMware quả URI

Thực hiện khai báo biến:

```
## vmware/roles/vmware-lab/vars/main.yml
---
## vars file for vmware
vcenter_hostname: vcenter.vlware.local
vcenter_username: administrator@vsphere.local
vcenter_password: Suncloud@2022
datacenter: VLWAREDC

validate_certs: no
```

Playbook:
```
## vmware/roles/vmware-lab/tasks/6.uri-login-vmware.yml
---
- name: Gather cluster information for vCenter
  gather_facts: no
  vars_files:
    - ../vars/main.yml
  hosts: localhost
  tasks:
    - name: Login into vCenter
      uri:
        url: https://{{ vcenter_hostname }}/api/session
        force_basic_auth: yes
        validate_certs: "{{ validate_certs }}"
        method: POST
        status_code: 201
        user: "{{ vcenter_username }}"
        password: "{{ vcenter_password }}"
      register: login

    - name: debug message
      debug:
        var: login
```

Kết quả:

```
root@ansible:~/Ansible-note# ansible-playbook vmware/roles/vmware-lab/tasks/6.uri-login-vmware.yml

PLAY [Gather cluster information for vCenter] *****************************************************

TASK [Login into vCenter] ***********************************************
ok: [localhost]

TASK [debug message] **********************************************
ok: [localhost] => {
    "login": {
        "changed": false,
        "connection": "close",
        "content_type": "application/json",
        "cookies": {},
        "cookies_string": "",
        "date": "Mon, 03 Apr 2023 14:09:05 GMT",
        "elapsed": 0,
        "failed": false,
        "json": "9bfec177674a50f101cc8d9cd46166c4",
        "msg": "OK (unknown bytes)",
        "redirected": false,
        "server": "envoy",
        "status": 201,
        "transfer_encoding": "chunked",
        "url": "https://vcenter.vlware.local/api/session",
        "vmware_api_session_id": "9bfec177674a50f101cc8d9cd46166c4",
        "x_envoy_upstream_service_time": "405"
    }
}

PLAY RECAP ********************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


```
## 2.2 Sử dụng URI lấy thông tin host


```
---
- name: Gather cluster information for vCenter
  gather_facts: no
  vars_files:
    - ../vars/main.yml
  hosts: localhost
  tasks:
    - name: Login into vCenter
      uri:
        url: https://{{ vcenter_hostname }}/api/session
        force_basic_auth: yes
        validate_certs: "{{ validate_certs }}"
        method: POST
        status_code: 201
        user: "{{ vcenter_username }}"
        password: "{{ vcenter_password }}"
      register: login
      
    - name: debug message
      debug:
        var: login

    - name: Gather hosts info using REST API
      uri:
        url: "https://{{ vcenter_hostname }}/api/vcenter/host"
        force_basic_auth: yes
        method: GET
        validate_certs: no
        headers:
          vmware-api-session-id: "{{ login.json }}"
      register: hosts_info
    - name: Print hosts info
      debug:
          var: hosts_info
```

Kết quả:

```

root@ansible:~/Ansible-note# ansible-playbook /root/Ansible-note/vmware/roles/vmware-lab/tasks/7.get_host.yml

PLAY [Gather cluster information for vCenter] ******************************************************************************************************************************************

TASK [Login into vCenter] **************************************************************************************************************************************************************
ok: [localhost]

TASK [debug message] *******************************************************************************************************************************************************************
ok: [localhost] => {
    "login": {
        "changed": false,
        "connection": "close",
        "content_type": "application/json",
        "cookies": {},
        "cookies_string": "",
        "date": "Mon, 03 Apr 2023 14:16:20 GMT",
        "elapsed": 0,
        "failed": false,
        "json": "3b4b0d5f8d8bcad0e046292dfc351081",
        "msg": "OK (unknown bytes)",
        "redirected": false,
        "server": "envoy",
        "status": 201,
        "transfer_encoding": "chunked",
        "url": "https://vcenter.vlware.local/api/session",
        "vmware_api_session_id": "3b4b0d5f8d8bcad0e046292dfc351081",
        "x_envoy_upstream_service_time": "367"
    }
}

TASK [Gather hosts info using REST API] ************************************************************************************************************************************************
ok: [localhost]

TASK [Print hosts info] ****************************************************************************************************************************************************************
ok: [localhost] => {
    "hosts_info": {
        "changed": false,
        "connection": "close",
        "content_type": "application/json",
        "cookies": {},
        "cookies_string": "",
        "date": "Mon, 03 Apr 2023 14:16:22 GMT",
        "elapsed": 0,
        "failed": false,
        "json": [
            {
                "connection_state": "CONNECTED",
                "host": "host-14",
                "name": "10.0.11.11",
                "power_state": "POWERED_ON"
            }
        ],
        "msg": "OK (unknown bytes)",
        "redirected": false,
        "server": "envoy",
        "status": 200,
        "transfer_encoding": "chunked",
        "url": "https://vcenter.vlware.local/api/vcenter/host",
        "x_envoy_upstream_service_time": "454"
    }
}

PLAY RECAP *****************************************************************************************************************************************************************************
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```


## 2.3
Lấy thông thông tin vm thông qua uri

```
## GET VM 
---
- name: Gather cluster information for vCenter
  gather_facts: no
  vars_files:
    - ../vars/main.yml
  hosts: localhost
  tasks:
    - name: Login into vCenter
      uri:
        url: https://{{ vcenter_hostname }}/api/session
        force_basic_auth: yes
        validate_certs: "{{ validate_certs }}"
        method: POST
        status_code: 201
        user: "{{ vcenter_username }}"
        password: "{{ vcenter_password }}"
      register: login
      
    - name: debug message
      debug:
        var: login

    - name: Gather hosts info using REST API
      uri:
        url: "https://{{ vcenter_hostname }}/api/vcenter/vm"
        force_basic_auth: yes
        method: GET
        validate_certs: no
        headers:
          vmware-api-session-id: "{{ login.json }}"
      register: vms_info
    - name: Print hosts info
      debug:
          var: vms_info
```

Kết quả

```
root@ansible:~/Ansible-note# ansible-playbook /root/Ansible-note/vmware/roles/vmware-lab/tasks/8.get_vms.yml

PLAY [Gather cluster information for vCenter] ******************************************************************************************************************************************

TASK [Login into vCenter] **************************************************************************************************************************************************************
ok: [localhost]

TASK [debug message] *******************************************************************************************************************************************************************
ok: [localhost] => {
    "login": {
        "changed": false,
        "connection": "close",
        "content_type": "application/json",
        "cookies": {},
        "cookies_string": "",
        "date": "Mon, 03 Apr 2023 14:28:45 GMT",
        "elapsed": 0,
        "failed": false,
        "json": "f75a1bac94a89f96470bb336447c4e48",
        "msg": "OK (unknown bytes)",
        "redirected": false,
        "server": "envoy",
        "status": 201,
        "transfer_encoding": "chunked",
        "url": "https://vcenter.vlware.local/api/session",
        "vmware_api_session_id": "f75a1bac94a89f96470bb336447c4e48",
        "x_envoy_upstream_service_time": "460"
    }
}

TASK [Gather hosts info using REST API] ************************************************************************************************************************************************
ok: [localhost]

TASK [Print hosts info] ****************************************************************************************************************************************************************
ok: [localhost] => {
    "vms_info": {
        "changed": false,
        "connection": "close",
        "content_type": "application/json",
        "cookies": {},
        "cookies_string": "",
        "date": "Mon, 03 Apr 2023 14:28:46 GMT",
        "elapsed": 0,
        "failed": false,
        "json": [
            {
                "cpu_count": 2,
                "memory_size_MiB": 4096,
                "name": "LAB-U20-SRV01",
                "power_state": "POWERED_ON",
                "vm": "vm-1005"
            },
            {
                "cpu_count": 8,
                "memory_size_MiB": 16384,
                "name": "LAB-U20-SRV03",
                "power_state": "POWERED_ON",
                "vm": "vm-1009"
            },
            {
                "cpu_count": 4,
                "memory_size_MiB": 8192,
                "name": "LAB-C7-SRV01",
                "power_state": "POWERED_OFF",
                "vm": "vm-1010"
            },
            {
                "cpu_count": 4,
                "memory_size_MiB": 8192,
                "name": "LAB-U20-AWX",
                "power_state": "POWERED_ON",
                "vm": "vm-1014"
            },
            {
                "cpu_count": 4,
                "memory_size_MiB": 8192,
                "name": "LAB-U20-AnsibleOSS",
                "power_state": "POWERED_ON",
                "vm": "vm-1015"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 8192,
                "name": "LAB-U20-GITLAB",
                "power_state": "POWERED_ON",
                "vm": "vm-1017"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 6144,
                "name": "lcm",
                "power_state": "POWERED_ON",
                "vm": "vm-2002"
            },
            {
                "cpu_count": 8,
                "memory_size_MiB": 16384,
                "name": "idm",
                "power_state": "POWERED_ON",
                "vm": "vm-2003"
            },
            {
                "cpu_count": 12,
                "memory_size_MiB": 43008,
                "name": "vra",
                "power_state": "POWERED_ON",
                "vm": "vm-2004"
            },
            {
                "cpu_count": 4,
                "memory_size_MiB": 8192,
                "name": "vrli",
                "power_state": "POWERED_ON",
                "vm": "vm-2005"
            },
            {
                "cpu_count": 4,
                "memory_size_MiB": 8192,
                "name": "LAB-MINIO-01",
                "power_state": "POWERED_OFF",
                "vm": "vm-2011"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "LAB-MINIO-02",
                "power_state": "POWERED_OFF",
                "vm": "vm-2012"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "LAB-MINIO-03",
                "power_state": "POWERED_OFF",
                "vm": "vm-2013"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "LAB-MINIO-04",
                "power_state": "POWERED_OFF",
                "vm": "vm-2014"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "LAB-U20-HARBOR",
                "power_state": "POWERED_ON",
                "vm": "vm-2015"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "vm-netbox01",
                "power_state": "POWERED_OFF",
                "vm": "vm-2038"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "vm-netbox02",
                "power_state": "POWERED_OFF",
                "vm": "vm-2039"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "Lab-Ansible-1",
                "power_state": "POWERED_ON",
                "vm": "vm-2056"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "lab-Netbox",
                "power_state": "POWERED_ON",
                "vm": "vm-2060"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "RedisSentinel01",
                "power_state": "POWERED_OFF",
                "vm": "vm-2062"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "RedisSentinel02",
                "power_state": "POWERED_OFF",
                "vm": "vm-2063"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "RedisSentinel03",
                "power_state": "POWERED_OFF",
                "vm": "vm-2064"
            },
            {
                "cpu_count": 1,
                "memory_size_MiB": 2048,
                "name": "HuyDV-Centos7-Client1",
                "power_state": "POWERED_ON",
                "vm": "vm-2068"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "LAB-U20-NETBOX-CTT2",
                "power_state": "POWERED_ON",
                "vm": "vm-2128"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "Cloud_Machine_1-mcm459-228418891002",
                "power_state": "POWERED_OFF",
                "vm": "vm-2162"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "Cloud_Machine_1-mcm486-228450227930",
                "power_state": "POWERED_OFF",
                "vm": "vm-2171"
            },
            {
                "cpu_count": 8,
                "memory_size_MiB": 40960,
                "name": "labccie",
                "power_state": "POWERED_ON",
                "vm": "vm-2190"
            },
            {
                "cpu_count": 16,
                "memory_size_MiB": 4096,
                "name": "WIN-tmp-test",
                "power_state": "POWERED_OFF",
                "vm": "vm-2191"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 2048,
                "name": "LAB-U20-Checkmk",
                "power_state": "POWERED_ON",
                "vm": "vm-2192"
            },
            {
                "cpu_count": 1,
                "memory_size_MiB": 2048,
                "name": "LIN-SERVER-02",
                "power_state": "POWERED_ON",
                "vm": "vm-2198"
            },
            {
                "cpu_count": 1,
                "memory_size_MiB": 2048,
                "name": "LIN-SERVER-03",
                "power_state": "POWERED_OFF",
                "vm": "vm-2199"
            },
            {
                "cpu_count": 1,
                "memory_size_MiB": 2048,
                "name": "LIN-SERVER-01",
                "power_state": "POWERED_ON",
                "vm": "vm-2200"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 4096,
                "name": "Windows-jump",
                "power_state": "POWERED_OFF",
                "vm": "vm-25"
            },
            {
                "cpu_count": 1,
                "memory_size_MiB": 2048,
                "name": "LAB-VYOS",
                "power_state": "POWERED_ON",
                "vm": "vm-26"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 8192,
                "name": "LAB-PROXY",
                "power_state": "POWERED_ON",
                "vm": "vm-29"
            },
            {
                "cpu_count": 4,
                "memory_size_MiB": 8192,
                "name": "LAB-JUM-WIN01",
                "power_state": "POWERED_ON",
                "vm": "vm-31"
            },
            {
                "cpu_count": 4,
                "memory_size_MiB": 16384,
                "name": "LAB-BOOTSTRAP01",
                "power_state": "POWERED_OFF",
                "vm": "vm-34"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 6144,
                "name": "LAB-AD01",
                "power_state": "POWERED_ON",
                "vm": "vm-36"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 12288,
                "name": "LAB-VCENTER-VLWARE",
                "power_state": "POWERED_ON",
                "vm": "vm-38"
            },
            {
                "cpu_count": 1,
                "memory_size_MiB": 128,
                "name": "vCLS-3c1ae833-0a2d-4c79-8493-f30247e7f86d",
                "power_state": "POWERED_ON",
                "vm": "vm-39"
            },
            {
                "cpu_count": 1,
                "memory_size_MiB": 2048,
                "name": "LAB-UBUNTU-VM01",
                "power_state": "POWERED_OFF",
                "vm": "vm-46"
            },
            {
                "cpu_count": 1,
                "memory_size_MiB": 2048,
                "name": "LAB-UBUNTU-VM02",
                "power_state": "POWERED_OFF",
                "vm": "vm-47"
            },
            {
                "cpu_count": 4,
                "memory_size_MiB": 8192,
                "name": "LAB-JUM-UBUNTU01",
                "power_state": "POWERED_OFF",
                "vm": "vm-48"
            },
            {
                "cpu_count": 2,
                "memory_size_MiB": 4096,
                "name": "LAB-PRITUNL",
                "power_state": "POWERED_ON",
                "vm": "vm-49"
            },
            {
                "cpu_count": 4,
                "memory_size_MiB": 8192,
                "name": "LAB-JUMP-14-225-16-167",
                "power_state": "POWERED_ON",
                "vm": "vm-87"
            },
            {
                "cpu_count": 4,
                "memory_size_MiB": 8192,
                "name": "PRO-JUMP-14-225-16-167",
                "power_state": "POWERED_ON",
                "vm": "vm-88"
            }
        ],
        "msg": "OK (unknown bytes)",
        "redirected": false,
        "server": "envoy",
        "status": 200,
        "transfer_encoding": "chunked",
        "url": "https://vcenter.vlware.local/api/vcenter/vm",
        "x_envoy_upstream_service_time": "101"
    }
}

PLAY RECAP *****************************************************************************************************************************************************************************
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


```