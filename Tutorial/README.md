# Tìm hiểu Ansible cơ bản
1. [Cài đặt và sử dụng Ansible cơ bản](1.basic.md)
    * Tạo SSH đến được các host
    * Tạo được file Inventory cho các host
    * Ansible ping được đến các host
1. [Sử dụng AD-hoc Command để tương tác với các host](2.ad-hoc.md)
    * `ad-hoc command` dùng để tương tác trực tiếp với các host thông qua câu lệnh trực tiếp (Còn kiểu tương tác khác là tương tác qua play book).
1. [Hướng dẫn cách sử dụng Playbook cơ bản](3.playbook.md)
    * Playbook là có nhiều task bên trong. Mỗi task thực hiện 1 tác vụ riêng. Chạy playbook là thực hiện chạy tuần tự các task bên trong playbook. 
1. [Hướng dẫn sử dụng các module trong Ansible](4.module.md)
    * Đây là các module cơ bản được Ansible sử dụng(Giống như thực hiện với linux: tạo file xóa file, cài đặt gói ...)
    * Đối với mỗi nhiệm vụ cần có 1 module tương ứng để có thể thực hiện task.
1. [Loop - Sử dụng vòng lặp trong Ansible](5.loop.md)
1. [Fact](6.fact.md)
    * Ansible facts là dữ liệu được thu thập từ các nút mục tiêu và được trả lại kết quả về cho các nút controller.
1. [When](7.when.md)
1. [Template](8.template.md)
1. []()
1. []()