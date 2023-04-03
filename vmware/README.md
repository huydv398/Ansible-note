# Ansible-VMware Part 2

```
~/Ansible-VMware/
├── ansible.cfg
├── inventory #nơi khai báo các tham số và biến số chung
│   ├── group_vars
│   │   └── all
│   │       ├── 01-dns_vars.yml
│   │       ├── 02-deploy_ova_vars.yml
│   │       ├── 03-windows_default_vars.yml
│   │       └── 04-vcenter_server.yml
│   ├── hosts
│   └── host_vars
│       ├── lin-server-01.yml #khai báo thông số chi tiết cho từng máy ảo. 
│       ├── lin-server-02.yml #dùng để khởi tạo máy ảo. khi thực hiện chạy playbook.
│       ├── lin-server-03.yml #~/Ansible-VMware/roles/vmware_create_virtual_machine/tasks/main.yml
│       ├── lin-server-04.yml
│       └── lin-server-05.yml
├── requirements.txt #Khai báo các gói cần thiết
├── roles # Khai báo các role
│   └── vmware_create_virtual_machine
│       ├── handlers
│       │   └── main.yml
│       ├── tasks
│       │   └── main.yml
│       └── vars
│           └── main.yml
├── ssh_config
├── vmware_create_linux_vm.yml
├── vmware_create_virtual_machines.yml
├── vmware_create_virtual_machine.yml
├── vmware_create_windows_vm.yml
└── vmware_deploy_ova.yml
```
## 1. Thực hiện cài cấu hình cơ bản
Thực hiện chạy lệnh sau để cài các gói cần thiết
```
pip install -r requirements.txt
```

Thực hiện sửa các tham số đăng nhập vào vcenter:
```
cat ~/vmware/inventory/group_vars/all/04-vcenter_server.yml

```
```
---
vcenter_hostname: 'vcenter.vlware.local'
vcenter_username: 'administrator@vsphere.local'
vcenter_password: 'Suncloud@2022'

vcenter_default_datacenter: VLWAREDC #Điền tên của Datacenter trên vcenter
vcenter_default_cluster: VLWARECL #Điền tên của Cluster trên vcenter
vcenter_default_folder: /
```

Khai báo host:
```
cat ~/vmware/inventory/hosts
```
## 2. Tạo 1 máy ảo (Không templeta)
Dùng để tạo từng máy ảo 1 theo thông số khai báo trực tiếp vào Playbook

File `vmware/roles/vmware-lab/tasks/create-vm.yml` có thông tin như sau:
```
- name: Create a New Virtual Machine
  vmware_guest:
    hostname: '{{ vcenter_hostname }}'
    username: '{{ vcenter_username }}'
    password: '{{ vcenter_password }}'
    datacenter: '{{ datacenter }}'
    validate_certs: no
    name: Linux_VM
    cluster: VLWARECL #tên cludter
    folder: /
    guest_id: centos7_64Guest
    disk:
    - size_gb: 20
      type: thin
      datastore: CTVN-lab #tên datasote
    hardware:
      memory_mb: 2048
      num_cpus: 1
      scsi: paravirtual
      hotadd_cpu: True
      hotadd_memory: True 
    networks:
    - name: "VM Network" #tên VLAN (Network)
      device_type: vmxnet3
#    state: poweredon
  delegate_to: localhost
```
```
# ansible-playbook vmware/lab-vmware.yml
```
```
oot@ansible:~/Ansible-note# ansible-playbook vmware/lab-vmware.yml 

PLAY [local] ***********************

TASK [Gathering Facts] **************************

TASK [vmware-lab : Create a New Virtual Machine] ******************
changed: [ansible -> localhost]

PLAY RECAP ***********
ansible                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
Bạn có thể thấy rằng quá trình thực thi đã thành công (ok=1) và một thay đổi đã được thực hiện (changed=1), có nghĩa là máy ảo đã được tạo. Nếu chúng ta xem vCenter, chúng ta có thể thấy máy ảo hiện đã tồn tại


Chú ý file host được khai báo như sau:
```
# cat /etc/ansible/hosts
[local]
ansible ansible_host=127.0.0.1 ansible_port=22 ansible_user=root
```
## 3. Tạo một máy ảo Linux từ một Template đã có sẵn
Bạn sẽ cần tạo một Template cho máy ảo trong vCenter, với các yêu cầu sau:

* Đã cài đặt phiên bản mới nhất của VMware tools; Tôi khuyên bạn nên sử dụng `open-vm-tools`, được bao gồm trong kho lưu trữ cho hầu hết các bản phân phối chính.
* Đảm bảo rằng PERL đã được cài đặt, nếu không việc tùy chỉnh sẽ không thành công.

Một play sẽ có thông tin như sau:

Play là file: `vmware/roles/vmware-lab/tasks/create-vm-tmp.yml` nhưng sử dụng role để gọi ra bên ngoài tại file: `vmware/lab-vmware.yml`

```
##vmware/lab-vmware.yml
# cat vmware/lab-vmware.yml
---
- hosts: local
  roles:
  - vmware-lab #chạy playbook ~/vmware/roles/vmware-lab/tasks/main.yml

# cat vmware-lab/tasks/main.yml
---
 - import_tasks: create-vm-tmp.yml
```

```
## vmware/roles/vmware-lab/tasks/create-vm-tmp.yml
  - name: Create a Linux Virtual Machine
    vmware_guest:
      hostname: '{{ vcenter_hostname }}'
      username: '{{ vcenter_username }}'
      password: '{{ vcenter_password }}'
      datacenter: '{{ datacenter }}'
      validate_certs: no
      name: c7-tmp-test3 #Đặt tên cho VM
      cluster: VLWARECL #Tên Cluster
      folder: /
      template: CentOS-7 #Tên Template muốn clone
      datastore: CTVN-lab # Tên Datastore
      hardware:
        memory_mb: 4096
        num_cpus: 2
        scsi: paravirtual
      disk:
        - size_gb: 20
          type: thin
      networks: #đặt IP Static
        - name: VLAN-21-MGNT
          ip: 10.0.11.130 
          netmask: 255.255.255.0
          gateway: 10.0.11.1
          domain: vlware.local
          dns_servers:
          - 8.8.8.8
          - 1.1.1.1
      customization:
        hostname: tmp3 #hostname sẽ đặt cho VM
      wait_for_customization: yes
      wait_for_ip_address: True
      state: poweredon
    delegate_to: localhost
```

Thực hiện chạy playbook:
```
ansible-playbook vmware/lab-vmware.yml
```
## 4. Tạo một máy ảo Window từ một Template đã có sẵn
Triển khai máy ảo từ mẫu Windows sử dụng cách chơi tương tự như triển khai Linux, nhưng cho phép một số tùy chỉnh bổ sung, chẳng hạn như thêm máy ảo vào Active Directory và đặt mật khẩu quản trị viên.

Play là file: `vmware/roles/vmware_create_virtual_machine/tasks/main.yml` nhưng sử dụng role để gọi ra bên ngoài tại file: `/vmware/vmware_create_virtual_machines.yml`

```
##vmware/lab-vmware.yml
# cat vmware/lab-vmware.yml 
---
- hosts: local
  roles:
  - vmware-lab #chạy playbook ~/vmware/roles/vmware-lab/tasks/main.yml

# cat vmware-lab/tasks/main.yml
---
 - import_tasks: create-vm-win-tmp.yml
```

```
## vmware/roles/vmware-lab/tasks/create-vm-win-tmp.yml
  - name: Create a Windows Virtual Machine
    vmware_guest:
      hostname: '{{ vcenter_hostname }}'
      username: '{{ vcenter_username }}'
      password: '{{ vcenter_password }}'
      datacenter: '{{ datacenter }}'
      validate_certs: no
      name: WIN-tmp-test #Đặt tên cho VM
      cluster: VLWARECL #Tên Cluster
      folder: /
      template: WIN16-TMP #Tên Template muốn clone
      datastore: CTVN-lab # Tên Datastore
      hardware:
        memory_mb: 4096
        num_cpus: 2
        scsi: paravirtual
      networks: #đặt IP Static
        - name: VLAN-21-MGNT
          ip: 10.0.11.130 
          netmask: 255.255.255.0
          gateway: 10.0.11.1
          domain: vlware.local
          dns_servers:
          - 8.8.8.8
          - 1.1.1.1
      customization:
        password: Suncloud@2022!
        dns_servers:
        - 8.8.8.8
        - 10.0.11.254
        timezone: 205 #TIMEZONE VIETNAM 205
        hostname: tmp3 #hostname sẽ đặt cho VM
      wait_for_customization: yes
      wait_for_ip_address: True #play sẽ đợi vCenter dò tìm địa chỉ IP trên máy ảo trước khi hoàn tất.
      state: poweredon
    delegate_to: localhost
```

Thực hiện chạy playbook:
```
ansible-playbook vmware/lab-vmware.yml
```

>**Chú ý**: Đối với tạo window thì chưa cho fix tham số disk

## 5 Thực hiện thay đổi thông số máy ảo
Điều tuyệt vời về Ansible là nếu bạn chạy lại vở playbook này, nó sẽ không thử tạo một máy ảo khác. Thay vào đó, nó sẽ thoát ra với trạng thái OK, nếu nó phát hiện ra rằng máy ảo được chỉ định đã tồn tại và được bật nguồn.

Nhưng nếu chúng tôi thực hiện một số thay đổi đối với cấu hình mà chúng tôi muốn áp dụng cho máy ảo thì sao? Chà, Ansible sẽ chỉ thực hiện những thay đổi này đối với máy ảo nếu tham số `state: present` trong playbook. Ngoài ra, nếu bạn đang thực hiện thay đổi cấu hình cho phần cứng, thì máy ảo cũng có thể cần phải tắt nguồn trước (Ansible sẽ hiển thị lỗi nếu điều này là bắt buộc).

![image](https://prnt.sc/JYQjGBvQHirK)

Vì vậy, giả sử rằng máy ảo đã tắt và chúng tôi muốn kích hoạt hỗ trợ thêm nóng CPU và bộ nhớ. Chúng tôi chỉ cần thêm các cấu hình này trong phần phần cứng

## 6 Thực hiện tạo máy ảo từ 1 File OVA
??

## 7 Tạo Playbook triển khai có thể tái sử dụng lại các biến
Tận dụng các biến và các roles của ansible, đồng thời kết hợp để tạo ra nhiều máy ảo cùng lúc

Role được khai báo tại đường dẫn `~/vmware/roles/vmware_create_virtual_machine`:

```
---
- name: Create a New Virtual Machine
  vmware_guest:
    #thông tin vcenter được lấy từ file ~/vmware/inventory/group_vars/all/04-vcenter_server.yml
    hostname: "{{ vcenter_hostname }}"
    username: "{{ vcenter_username }}"
    password: "{{ vcenter_password }}"
    validate_certs: "{{ ova_validate_certs }}"
    datacenter: "{{ vcenter_default_datacenter }}"
    cluster: "{{ vcenter_default_cluster }}"
    folder: "{{ vcenter_default_folder }}"
    template: "{{ vm_guest_tmp }}" #Tên Template muốn clone
    datastore: "{{ vm_guest_tmp_Datastore }}" # Tên Datastore chứa TMP
    name: "{{ inventory_hostname_short | upper }}" # lấy tên từ file host ~//vmware/inventory/hosts
  # Các biến bên dưới được gọi ra từ ~/vmware/inventory/host_vars/vm-host
    guest_id: "{{ vm_guest_id }}"
    disk:
    - size_gb: "{{ vm_disk_size }}"
      type: "{{ vm_disk_type }}"
      datastore: "{{ vm_disk_datastore }}"
    hardware: 
      memory_mb: "{{ vm_hardware_memory_gb * 1024 }}"
      num_cpus: "{{ vm_hardware_cpu }}"
      scsi: "{{ vm_hardware_controller }}"
    networks: 
    - name: "{{ vm_network_label }}"
      device_type: "{{ vm_network_type }}"
      ip: "{{ vm_network_ip }}" 
      netmask: "{{ vm_network_netmask }}"
      gateway: "{{ vm_network_gateway }}"
      domain: "{{ vm_network_domain }}"
      dns_servers: "{{ vm_network_dns }}"
    customization:
      hostname: "{{ vm_network_hostname }}" #hostname sẽ đặt cho VM
    wait_for_customization: yes # tự động bật máy ảo khi hoàn thành
    wait_for_ip_address: True #Đợi khi VM nhận IP máy ảo. sẽ đợi vCenter dò tìm địa chỉ IP trên máy ảo trước khi hoàn tất.
    state: poweredon #Thực hiện bật máy ảo sau khi tạo
  delegate_to: localhost
```

Ở đây, hầu hết bạn có thể thấy rằng tôi gọi vào từ các biến `"{{ var}}"`. Vì roles trong ansible cần tuân thủ một số các quy tắc theo mức độ ưu tiên. Tôi khai báo ở 2 nơi:
* `inventory/group_vars/all`: các thông tin khác sử dụng chung vcenter, dns, windows_domain
* `inventory/host_vars`: gồm các thông tin chi tiết cho máy ảo. Tất cả các máy chủ muốn tạo được đặt tại đây:
```
---
vm_guest_id: centos7_64Guest

# VM Hardware
vm_hardware_memory_gb: 2
vm_hardware_cpu: 1
vm_hardware_controller: paravirtual

# VM Disk
vm_disk_size: 20
vm_disk_type: thin
vm_disk_datastore: CTVN-lab

# VM Network
vm_network_label: VLAN-21-MGNT
vm_network_type: vmxnet3
vm_network_ip: 10.0.11.81
vm_network_netmask: 255.255.255.0
vm_network_gateway: 10.0.11.1
vm_network_domain: vlware.local
vm_network_dns: 8.8.8.8
vm_network_hostname: test-tmp1

#template
vm_guest_tmp: CentOS-7
vm_guest_tmp_Datastore: CTVN-lab
```

`inventory/hosts`: tập hợp các nhóm host cho ansible gọi đến khi thực hiện playbook. 
```
...
[linux]
lin-server-01
lin-server-02
lin-server-03
lin-server-04
lin-server-05
...
```
Trong nhóm `linux` có 5 máy ảo, triển khai các máy ảo bằng playbook, tôi có thể gọi lệnh `ansible-playbook`.

```
ansible-playbook ~/vmware/vmware_create_virtual_machines.yml -i ~/vmware/inventory/hosts
```

Sau khi thực thi kết quả trả về như sau:

![image](https://prnt.sc/F-xlur22BJ1y)

![image](https://prnt.sc/KYFnrheg4c0a)

Điều tuyệt vời về Ansible là nó sẽ thực thi những điều này song song.

Tài liệu tham khảo: 
* https://www.simplygeek.co.uk/2019/07/18/automate-vsphere-virtual-machine-and-ova-appliance-deployments-using-ansible/#Create_a_New_Virtual_Machine_no_template
