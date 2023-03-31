Role Name
=========

Role này dùng để thực hiện tạo hàng loạt các máy ảo được khai báo ở thư mục: `/~/vmware/inventory/host_vars`
Requirements
------------

Chú ý chỉnh sửa các tham số của máy ảo tại thư mục `~/vmware/inventory/host_vars`

Role Variables
--------------

Mô tả về các biến có thể thiết lập cho vai trò này sẽ có ở đây, bao gồm bất kỳ biến nào trong defaults/main.yml, vars/main.yml và bất kỳ biến nào có thể/nên được đặt thông qua tham số cho vai trò. Bất kỳ biến nào được đọc từ các vai trò khác và/hoặc phạm vi toàn cầu (ví dụ: hostvars, group vars, v.v.) cũng nên được đề cập ở đây.

Dependencies
------------

Danh sách các vai trò khác được lưu trữ trên Galaxy sẽ có tại đây, cùng với mọi chi tiết liên quan đến các tham số có thể cần được đặt cho các vai trò khác hoặc các biến được sử dụng từ vai trò khác

Example Playbook
----------------

Bao gồm một ví dụ về cách sử dụng vai trò của bạn (ví dụ: với các biến được truyền dưới dạng tham số) cũng luôn tốt cho người dùng::

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

Phần tùy chọn dành cho tác giả vai trò bao gồm thông tin liên hệ hoặc trang web (HTML không được phép).
