# Ảo hóa

## Overview
### 1. What is virtualization.
Ảo hóa là công nghệ được thiết kế để tạo ra tầng trung gian giữa hệ thống phần cứng máy tính và phần mềm chạy trên nó. Ý tưởng của công nghệ ảo hóa máy chủ là từ một máy vật lý đơn lẻ có thể tạo thành nhiều máy ảo độc lập. Mỗi một máy ảo đều có một thiết lập nguồn hệ thống riêng rẽ, hệ điều hành riêng và các ứng dụng riêng.

Virtualization có thể xảy ra ở tất cả các thành phần vật lý như là: network, storage, application, access, ...
Ví dụ:
	* SDN(Software Defined Network): Network Virtualization.
	* SDS(Software Defined Storage): Storage Virtualization.
	* Các ứng dụng streaming, remove desktop và các công nghệ virtualization khác đều được gọi là Application Virtualization

Những giải pháp ảo hóa Linux phổ biến bao gồm: KVM, XEN, QEMU và VirtualBox.

### 2. Tại sao tôi nên dùng Linux Virtualization?
* Được hỗ trợ mạnh mẽ từ cộng đồng mỗi khi gặp vấn đề.
* Cung cấp những giải pháp tốt về infrastructure.
* Mã nguồn được public nên đảm bảo được tính bảo mật và mềm dẻo.
* Hiệu năng tốt.

### 3. Ưu điểm của ảo hóa.
* **Hợp nhất Server**: Ảo hóa giúp tiết kiệm chi phí vận hành. Làm giảm số lượng thiết bị vật lý. Tiết kiệm không gian và năng lượng. Tận dụng được nhiều hơn khả năng của phần cứng.
* **Service độc lập**: Dễ dàng tạo ra được sự độc lập giữa các Service của máy thật và Service của máy ảo. 
* **Cung cấp 1 Server nhanh hơn**: Server dễ dàng được tạo ra thay vì phải setup phần cứng mới. 
* **Khả năng khôi phục**: Dễ dàng tạo ra một bản snapshot khi có cập nhật. Từ đó dễ dàng phục hồi lại. Ngoài ra còn dễ dàng trong việc di chuyển trực tuyến và ngoại tuyến.

* **Cân bằng tải động**: Tùy thuộc vào chính sách mà bạn đặt. Khi khối lượng công việc của máy chủ thay đổi, ảo hóa cung cấp khả năng cho các máy ảo đang sử dụng quá mức tài nguyên của máy chủ, sẽ được chuyển (di chuyển trực tiếp) sang các máy chủ không được sử dụng, dựa trên các chính sách bạn đặt. Hầu hết các giải pháp ảo hóa đều đi kèm với các chính sách như vậy cho người dùng. Cân bằng tải động này tạo ra việc sử dụng hiệu quả tài nguyên máy chủ.


* **Phát triển và test nhanh hơn**: Dễ dàng tạo môi trường để test và thử nghiệm hệ thống trong ảo hóa. Giúp giảm khả năng lỗi tương tắc giữa các vấn đề không liên quan từ các ứng dụng khác. 

* **Cải thiện độ tin cậy và bảo mật hệ thống**: Có thể dễ dàng cô lập và cấu hình mạng cho máy ảo giúp hệ thống bảo mật hơn. 

### 4. Ảo hóa và phân vùng hệ thống.
**Ring 0**:  Kernel
- Tương tác trực tiếp với CPU, memory, I/O Ports. 
- Ở vòng này thì CPU chạy ở chế độ Kernel mode.
- Hypervisor/ Virtual Machine Monitor(VMM) được chạy ở ring 0
**Ring 1 và 2**: Device Drivers
- Vòng này hầu hết không sử dụng.
- GuestOS chạy ở ring 1.
**Ring 3**: Applications
- CPU chạy ở chế độ User mode. 
- Tất cả các ứng dụng chạy ở vòng này.

**User mode**: Chỉ cho phép những lệnh cần thiết để tính toán và xử lý dữ liệu. Các ứng dụng chạy ở mode này và chỉ sử dụng phần cứng bằng cách thông qua kernel bằng lời gọi hệ thống(system call).
**Kernel mode**: Cho phép chạy đầy đủ tập lệnh CPU, bao gồm cả các lệnh đặc quyền. Điều tất nhiên là mode này chỉ dành cho hệ điều hành chạy.

Như thiết kế ban đầu thì các hệ điều hành chạy trên Ring 0 thì nay bị đẩy lên Ring 1. Vấn đề là trong hệ điều hành có một số lệnh chỉ có thể chạy được ở Ring 0. Để làm việc ở các lớp cao hơn, HĐH phải được viết lại để tránh các lệnh này. Nhưng điều này khó xảy ra, giải pháp xử lý những lệnh nhạy cảm này là:
* Binary translation.
* Paravirtualization.
* Hardware assisted virtualization.


## Full virtualization 
* Khi Guest muốn truy cập đến một số chức năng cả Host(Ví dụ như xem xét hiệu năng của toàn hệ thống) thì phải sử dụng **Binary Translation**. Việc sử dụng phương pháp này sẽ mất thời gian để biên dich qua lại => Hiệu suất ko dc tốt lắm.

## Paravirtualization
* Kernel của máy ảo được sửa đổi để có thể sử dụng API của VMM được trên Ring 0. Khi đó VMM thực hiện rồi trả về kết quả cho Guest.

## Hardware Assisted Virtualzation.
* Chíp của Intel và AMD nay đã được tích hợp Hardware Assisted Virtualzation. Chúng cho phép Guest chạy ở Ring 0. Lúc này VMM sẽ ở Ring -1 chỉ có tác dụng phân chia tài nguyên hệ thống và duy trì nhiều VM(suy luận là vậy).

## Các loại Hypervisor
- Hypervisor là 1 ứng dụng phần mềm chịu trách nhiệm chạy nhiều máy ảo trên 1 hệ thống . Nó chịu trách nhiệm tạo, duy trì, truy cập hệ thống.
### 1. Hypervisor chạy dưới lớp OS(Ring -1).
Ưu điểm: 
* Có hiệu suất cao vì tương tác trực tiếp với phần cứng. Gần với kernel và ở Ring 0 nên có thể thực thi được mọi thứ.
Nhược điểm:
* Khó cài đặt. 
Các dòng sản phẩm: Xen, Vmware ESXi, Microsoft Hyoer-V, Apple Boot Camp.

### 2. Hypervisor chạy trên lớp OS(Ring 1).
Lúc này Hypervisor chạy như 1 service.
Ưu điểm:
* Dễ cài đặt. 
* Phạm vi hỗ trợ phần cứng rộng vì có lớp OS quản lý.
Nhược điểm:
* Ít đặc quyền và có thể bị break.
* Không có khả năng truy cập vào phần cứng một cách trực tiếp => Hiệu xuất kém hơn.
Các dòng sản phẩm: Microsoft Virtual PC, Vmware Workstation, VMware Server, Vituabox.

### 3. Hypervisor chạy cùng lớp OS(Ring 0). 
Ưu điểm: 
* Dễ cài đặt hơn chạy dưới OS.
* Hiệu suất cao hơn Hypervisor chạy trên OS.
Nhược điểm: 
Các dòng sản phẩm: KVM 



