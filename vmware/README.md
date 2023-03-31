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
├── README.md
├── requirements.txt #Khai báo các gói cần thiết
├── roles # Khai báo các role
│   ├── vmware_create_virtual_machine
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── meta
│   │   │   └── main.yml
│   │   ├── README.md
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── tests
│   │   │   ├── inventory
│   │   │   └── test.yml
│   │   └── vars
│   │       └── main.yml
│   ├── vmware_deploy_linux_vm_from_template
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── meta
│   │   │   └── main.yml
│   │   ├── README.md
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── tests
│   │   │   ├── inventory
│   │   │   └── test.yml
│   │   └── vars
│   │       └── main.yml
│   └── vmware_deploy_windows_vm_from_template
│       ├── defaults
│       │   └── main.yml
│       ├── handlers
│       │   └── main.yml
│       ├── meta
│       │   └── main.yml
│       ├── README.md
│       ├── tasks
│       │   └── main.yml
│       ├── tests
│       │   ├── inventory
│       │   └── test.yml
│       └── vars
│           └── main.yml
├── ssh_config
├── vmware_create_linux_vm.yml
├── vmware_create_virtual_machines.yml
├── vmware_create_virtual_machine.yml
├── vmware_create_windows_vm.yml
└── vmware_deploy_ova.yml

26 directories, 43 files
```