# Ansible netbox note
Cài đặt netbox cho Ansible trước khi sử dụng
```
pip install pynetbox --upgrade
ansible-galaxy collection install netbox.netbox
```
## 1. Thực hiện login vào netbox
Muốn thao tác được với netbox cần đang nhập trước khi thực hiện các thao tác
```
---
- name: "GATHER DATA FROM NETBOX"
  hosts: netbox
  connection: local
  gather_facts: true
  vars:
    netbox_url: https://netbox-vlware.vn
    netbox_token: ca55aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaxxxx
```
## 2. Thực hiện tạo Site cho Netbox
Tạo một file var chứa tập chung các thông tin của netbox:
```
---
netbox_url: http://10.0.11.121
netbox_token: 151dsfasdfasdfasdfd6a0b8ff52f
region_DC: 'Mien Bac'
Site_group: VNPT
Site: IDC VNPT - NTL
Location_name: 
  - Phase1
  - Phase2
Rack: 
  - { Name: 'A1', Site: 'IDC VNPT - NTL', Location: 'Phase1' } 
  - { Name: 'A2', Site: 'IDC VNPT - NTL', Location: 'Phase1' } 
  - { Name: 'A3', Site: 'IDC VNPT - NTL', Location: 'Phase1' } 
  - { Name: 'B1', Site: 'IDC VNPT - NTL', Location: 'Phase2' } 
  - { Name: 'B2', Site: 'IDC VNPT - NTL', Location: 'Phase2' } 
  - { Name: 'B3', Site: 'IDC VNPT - NTL', Location: 'Phase2' } 
```

Thực hiên tạo task
```
- name: "Test NetBox module"
  connection: local
  hosts: localhost
  vars_files:
    - var.yml
  gather_facts: False
  tasks:
    - name: Create tenant within NetBox with only required information
      netbox.netbox.netbox_tenant:
        netbox_url: "{{ netbox_url }}"
        netbox_token: "{{ netbox_token }}"
        data: 
          name: huydv
        state: present  #absent || present

    
    #netbox.netbox.netbox_region
    - name: Create region within NetBox with only required information
      netbox.netbox.netbox_region:
        netbox_url: "{{ netbox_url }}"
        netbox_token: "{{ netbox_token }}"
        data:
          name: "{{ region_DC }}"
        state: present
    - name: Create site group within NetBox with only required information
      netbox.netbox.netbox_site_group:
        netbox_url: "{{ netbox_url }}"
        netbox_token: "{{ netbox_token }}"
        data:
          name: "{{ Site_group }}"
        state: present
    - name: Create site with all parameters
      netbox.netbox.netbox_site:
        netbox_url: "{{ netbox_url }}"
        netbox_token: "{{ netbox_token }}"
        data:
          name: "{{ Site }}"
          status: Active #Planned ||
          region: "{{ region_DC }}"
          site_group: "{{ Site_group }}"
          tenant: SUNCLOUD
          time_zone: Asia/Ho Chi Minh
          description: test description
          physical_address: Nam thang long
          slug: idc-vnpt-ntl
          comments: ### Placeholder
        state: present
    - name: Create location within NetBox with only required information
      netbox.netbox.netbox_location:
        netbox_url: "{{ netbox_url }}"
        netbox_token: "{{ netbox_token }}"
        data:
          name: "{{ item }}"
          site: "{{ Site }}"
        state: present
      loop: "{{ Location_name }}"
    - name: Create rack within NetBox with only required information
      netbox.netbox.netbox_rack:
        netbox_url: "{{ netbox_url }}"
        netbox_token: "{{ netbox_token }}"
        data:
          name: "{{ item.Name }}"
          site: "{{ item.Site }}"
          location: "{{ item.Location }}"
        state: present
      loop: "{{ Rack }}"
```

Sẽ tạo được như sau:
- 1 region: 'Mien Bac'
- 1 Site_group: VNPT
- 1 Site: IDC VNPT - NTL
- 2 Location: 
  - Phase1
  - Phase2
- 6 Rack: 
  - { Name: 'A1', Site: 'IDC VNPT - NTL', Location: 'Phase1' } 
  - { Name: 'A2', Site: 'IDC VNPT - NTL', Location: 'Phase1' } 
  - { Name: 'A3', Site: 'IDC VNPT - NTL', Location: 'Phase1' } 
  - { Name: 'B1', Site: 'IDC VNPT - NTL', Location: 'Phase2' } 
  - { Name: 'B2', Site: 'IDC VNPT - NTL', Location: 'Phase2' } 
  - { Name: 'B3', Site: 'IDC VNPT - NTL', Location: 'Phase2' } 

### 3. Thực hiện add device vào site-rack

Tôi thực hiện tạo 1 biến các chứa các thông tin cần thiết để thêm 1 thiết bị
```
  vars:
    Server:
    - { dev_name: "server 2", device_type: "dell_poweredge_r740", device_role: "server", dev_rack: "A1", position: "17"}
```
- dev_name: tên của thiết bị
- device_type: "dell_poweredge_r740", type đã tồn tại ở netbox
- device_role: "server", role đã tồn tại ở netbox
- dev_rack: "A1", vị trí thiết bị, đã tồn tại ở netbox
- position: "17", vị trí chưa có thiết bị, hoặc thay đổi nếu muốn thay đổi vị trí


```
- name: "add Divece modules"
  connection: local
  hosts: localhost
  gather_facts: False
  vars_files:
    - var.yml
  vars:
    Server:
    - { dev_name: "server 2", device_type: "dell_poweredge_r740", device_role: "server", dev_rack: "A1", position: "17"}
  tasks:
    - name: Create device within NetBox with only required information
      netbox.netbox.netbox_device:
        netbox_url: "{{ netbox_url }}"
        netbox_token: "{{ netbox_token }}"
        data:
          name: "{{ item.dev_name }}"
          device_type: "{{ item.device_type }}"
          device_role: server
          site: "{{ Site }}"
          rack: "{{ item.dev_rack }}"
          position: "{{ item.position }}"
          face: Front
        state: present
      loop: "{{ Server }}"
    
```

### 4. Thực hiện add máy ảo

```
##biến chưa các thông tin của máy ảo
---

# VM Hardware
vm_hardware_memory_gb: 16
vm_hardware_cpu: 8

# VM Disk
vm_disk_size: 1000
vm_disk_type: thin
vm_disk_datastore: CTVN-lab


#password vm
password_VM: Suncloud@2023!@#

#netbox
cluster_name: vmware-lab
tenant_name: SUNCLOUD

user_login: root
interface_name: 'Network Adapter 1'
prefix: "10.0.11.0/24"
netbox_ip: "10.0.11.83/24"
```

```
---
# tasks file for netbox-note-vm 
- name: Add a virtual machine to NetBox
  netbox.netbox.netbox_virtual_machine:
    netbox_url: "{{ netbox_url }}"
    netbox_token: "{{ netbox_token }}"
    data:
      name: "{{ inventory_hostname_short | upper }}"
      cluster: "{{ cluster_name }}"
      tenant: "{{ tenant_name }}"
      vcpus: "{{ vm_hardware_cpu }}"
      memory: "{{ vm_hardware_memory_gb }}"
      disk: "{{ vm_disk_size }}"
  # loop: "{{ target_vm_info }}"
  delegate_to: localhost

- name: Add an interface to the virtual machine
  netbox.netbox.netbox_vm_interface:
    netbox_url: "{{ netbox_url }}"
    netbox_token: "{{ netbox_token }}"
    data:
      virtual_machine: "{{ inventory_hostname_short | upper }}"
      name: "{{ interface_name }}"

  # loop: "{{ target_vm_info }}"
  delegate_to: localhost
- name: gắn IP vào máy ảo (NetBox 2.9+)
  netbox.netbox.netbox_ip_address:
    netbox_url: "{{ netbox_url }}"
    netbox_token: "{{ netbox_token }}"
    data:
      address: "{{ netbox_ip }}"
      assigned_object:
        name: "{{ interface_name }}"
        virtual_machine: "{{ inventory_hostname_short | upper }}"
      
    state: present
  delegate_to: localhost
```
