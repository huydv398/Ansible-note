# Ansible-VRLCM

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
# cat vmware/vmware_create_virtual_machines.yml 
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
# cat vmware/vmware_create_virtual_machines.yml 
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

Đối với tạo window thì chưa cho fix tham số disk

## 5 Thực hiện thay đổi thông số máy ảo
Điều tuyệt vời về Ansible là nếu bạn chạy lại vở playbook này, nó sẽ không thử tạo một máy ảo khác. Thay vào đó, nó sẽ thoát ra với trạng thái OK, nếu nó phát hiện ra rằng máy ảo được chỉ định đã tồn tại và được bật nguồn.

Nhưng nếu chúng tôi thực hiện một số thay đổi đối với cấu hình mà chúng tôi muốn áp dụng cho máy ảo thì sao? Chà, Ansible sẽ chỉ thực hiện những thay đổi này đối với máy ảo nếu tham số 'trạng thái' đã được đặt thành 'hiện tại' trong vở kịch. Ngoài ra, nếu bạn đang thực hiện thay đổi cấu hình cho phần cứng, thì máy ảo cũng có thể cần phải tắt nguồn trước (Ansible sẽ hiển thị lỗi nếu điều này là bắt buộc).

Vì vậy, giả sử rằng máy ảo đã tắt và chúng tôi muốn kích hoạt hỗ trợ thêm nóng CPU và bộ nhớ. Chúng tôi chỉ cần thêm các cấu hình này trong phần phần cứng: