# Ansible-note
## 1.  Ansible cơ bản
Tìm hiểu Ansible cơ bản qua các link sau:
### 1.1 Thực hiện cài đặt 
Cài đặt Ansible cơ bản trên Ubuntu 22 (Hoặc 20, 18 mà cứ bản mới nhất mà chơi). Cái này có thể lên háng để tìm cách cài chỉ cần chạy lệnh là nó tự cài thôi:
```
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
```
Trước mình tìm hiểu Ansible thì làm một lượt tài liệu này. 
* https://www.server-world.info/en/note?os=CentOS_7&p=ansible&f=1 

1. Lượt 1: cho nó chạy được đã. Xác định sử dụng các module để chạy. 
2. Lượt 2: mỗi lần làm thì seach module và tìm hiểu cách sử dụng của nó. 
3. Lượt 3: sâu chuỗi và tư duy cách mà nó kết hợp với nhau. 
4. Sau đó thì tìm thêm các tài liệu khác để đa dạng hướng dẫn hơn.
5. Khi đã hiểu hoặc seach module thì comment vào các task-module cho dễ nhớ

Các tài liệu có thể tham khảo
- [Link1](https://github.com/leucos/ansible-tuto) - [Link2](https://www.toptechskills.com/ansible-tutorials-courses/) - [Link3](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-20-04) - [Link4](https://mpolinowski.github.io/docs/category/ansible/) - [Link5](https://github.com/congto/VMUG20220727/blob/main/roles/netbox_activate_ip_addr/tasks/activate_ip_addr.yml) - [Link6](https://luxurious-preface-ad1.notion.site/ansible-vlware-http-uri-1010e24d7e3845e486a9879b7fc94f55) - [Link7](https://luxurious-preface-ad1.notion.site/ansible-vlware-24ffd704db3c40a9ad20b66370a75430) - [Link8](https://www.simplygeek.co.uk/2019/07/18/automate-vsphere-virtual-machine-and-ova-appliance-deployments-using-ansible/#Create_a_New_Virtual_Machine_no_template) - - [Link9](https://github.com/sky-joker/ansible-vmware-vm-inventory-report-generator)

## 2 [Thực hiện tìm hiểu Ansible cơ bản](Tutorial/README.md)
## 3 [Dùng Ansible với Netbox](netbox/readme.md)
## 4 [Dùng Ansible với VMware - Cơ bản ](vmware/roles/vmware-lab/README.md)
Sẽ có 2 các dùng với VMware là Native và URI. Phần này sẽ hướng dẫn triển khai kết nối tới VMware và lấy các thông tin qua Ansible

## 5 [Dùng Ansible với VMware - Nâng cao - Áp dụng vào triển khai thực tế ](vmware/README.md)
 Tại phần này sẽ giúp các bạn triển khai tạo máy ảo
## 6 [Dùng Ansible để gửi thông điệp về mail và telegram ](vmware/roles/send-alear/README.md)
## 7. Kết hợp từ phần 3 đến phần 6: Tạo máy ảo - Note netbox - Send alear


Thông tin máy ảo được note tại file `vmware/inventory/host_vars/[name server] `
```
---
vm_guest_id: ubuntu_64Guest

# VM Hardware
vm_hardware_memory_gb: 16
vm_hardware_cpu: 8
vm_hardware_controller: paravirtual

# VM Disk
vm_disk_size: 1000
vm_disk_type: thin
vm_disk_datastore: CTVN-lab

# VM Network
vm_network_label: DPG-VLAN21
vm_network_type: vmxnet3
vm_network_ip: 10.0.11.83
vm_network_netmask: 255.255.255.0
vm_network_gateway: 10.0.11.1
vm_network_domain: vlware.local
vm_network_dns: 8.8.8.8
# vm_network_hostname: test-tmp1

#template
vm_guest_tmp: TEMP-U20
vm_guest_tmp_Datastore: CTVN-lab

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

Ta cần 3 roles chính như sau:
```
# cat vmware/final-create-vm.yml
---
- hosts: linux
  become: no
  gather_facts: False
  roles:
  - vmware_create_virtual_machine #Tạo máy ảo
  - netbox-note-vm 
  - ansible-vmware-vm-inventory-report-generator # gửi thông tin về mail
```



