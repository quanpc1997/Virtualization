# Tìm hiểu monitor cơ bản
## I. Một số thuật ngữ cơ bản 
### 1. Linux metrics:
**Linux metrics** là từ chỉ chung các số liệu của các thành phần hệ thống như mạng, ram, cpu,... trong những thiết bị chạy linux.

### 2. CPU
**CPU Time** là lượng thời gian thực tế mà CPU thực thi một số lệnh. Trong quá trình chạy một số chương trình, CPU sẽ không hoạt động trong khi máy tính đang lấy dữ liệu từ bàn phím, gửi dữ liệu cho thiết bị cuối,... Do vậy thời gian sử dụng thực tế của CPU ít hơn nhiều thời gian thực hiện chương trình. Tận dụng lợi thế này, hệ điều hành đa nhiệm chia sẻ cho nhiều chương trình. 
**CPU Time** được sử dụng với nhiều mục đích khác nhau. Có thể so sánh hiệu năng giữa 2 CPU, đo đạc lượng thời gian xử lý được phân bố cho các chương trình khác nhau trong môi trường đa nhiệm, ...

**CPU_iowait** Khoảng thời gian của một ứng dụng khi CPU nhàn rỗi vì phải chờ nhập xuất dữ liệu.
IOWait rất quan trọng vì nó thường là một số liệu chính để biết liệu bạn có bị tắc nghẽn trên IO hay không. Nhưng sự vắng mặt của iowait không có nghĩa là ứng dụng của bạn không bị tắc nghẽn trên IO. Giả sử có 2 ứng dụng, 1 ứng dụng bị ngẽn, một ứng dụng ngốn CPU nặng và %CPU + %System xấp xỉ 100 thì iowait vẫn hiện bằng 0.
Để phát hiện iowait ta có thể sử dụng lệnh **iostat** và **strace** tôi sẽ trình bày ở dưới.
**Nguồn: https://serverfault.com/questions/12679/can-anyone-explain-precisely-what-iowait-is**

### 3. RAM 
**RAM free**(Ram trống) là lượng ram trống chưa được sử dụng. Có thể sử dụng chúng bất cứ lúc nào. 
**RAM available**(Ram khả dụng) bao gồm cả lượng ram trống_(RAM free)_ và bộ nhớ đệm_(cache)_. 

### 4. Cache 
**Cache** là bộ nhớ đệm thường được gắn trực tiếp lên CPU để lưu giữ lại data thường được sử dụng . Làm giảm việc truy cập đến storage có tốc độ chậm hơn. Tốc độ truy suất trên cache nhanh hơn RAM 

Người ta sử dụng thuật toán Cache Algorithms để hướng dẫn và duy trì bộ nhớ đệm cache. Bằng cách nắm được những ứng dụng thường xuyên nhất, gần đây nhất thì đưa lên đầu. Các ứng dụng ít sử dụng thường bị đẩy dần xuống cuối và cuối cùng là bị xóa.

Khi chúng ta duyệt web. Mỗi khi ta chuyển hướng đến một tài nguyên mới thì cache sẽ lưu lại. Bằng chứng là khi ta nhấn "Back" thì dữ liệu trước đó sẽ được hiện lên một cách nhanh chóng.

Có 3 loại bộ nhớ cache: 
	* **Write-around cache**	:
	* **Write-through cache**	: Cache ghi đè dữ liệu cả vào bộ nhớ cache và bộ nhớ storage. 
		* Nhược điểm: Ghi chậm. 
	* **Write-back cache**		: Khác với Write-through cache ở chỗ nó lưu vào cache trước rồi từ cách lưu vào storage.
		* Nhược điểm: Do bị khác biệt tốc độ nên dữ liệu dễ bị mất khi storage chưa kịp lưu mà Cache đã phải lưu rồi.

Cache có thể được sử dụng như một Server(Proxy Server) , Disk cache(VD: Swap trên Linux), Ram Cache, Flash Cache.

**(*)** Ram cũng có bộ nhớ Cache. Tốc độ Cache của Ram nhỏ hơn của CPU 1 phần vì xa CPU hơn. 

Tham khảo: https://tech.vccloud.vn/cache-bo-nho-dem-la-gi-vai-tro-va-phan-loai-cache-20180618111100714.htm

### 5. Buffer 
**Buffer** là một vùng nhớ vật lý tạm thời để chứa data tạm trước khi được chuyển đến một nơi khác. Ví dụ 
**Buffer** là tầng chung gian để truyền dữ liệu giữa 2 Process. Việc này xảy ra khi giữa 2 Process chưa đồng bộ hóa về mặt tốc độ. 

**(*)** Sự khác nhau giữa Cache và Buffer ở mục đích của chúng: 
**Cache** chủ yếu để truy xuất nhanh. 
**Buffer** là tầng ở giữa làm giảm sự khác biệt về tốc độ giữa 2 Process.
_Ví dụ như thao tác ghi dữ liệu down về từ trên Internet vào đĩa cứng. Khi tốc độ mạng nhanh hơn tốc độ ghi của đĩa thì nó sẽ được chuyển vào buffer và ghi dần. Hoặc trong các gói tin trong mạng, khi có một lượng gói tin lớn mà router chưa kịp xử lý hết thì các gói tin đó sẽ được xếp vào buffer để xử lý dần, ..._

### 6. Storage 
**DIsk Free** là phần dung lượng đĩa sẵn sàng để ghi dữ liệu mà không ảnh hưởng đến các dữ liệu khác.

**IOPS** _(Input Outputs Per Second)_ là môt đơn vị chuẩn để đánh giá hiệu năng của các thiết bị lưu trữ như đĩa cứng SATA, SAS và SSD. Nó được tính bằng số lần đọc ghi tối đa trên 1 s. IOPS càng cao thì hiệu năng càng cao. 


### 7. Network 
**Letancy** là độ chễ trung bình trong một đơn vị thời gian.
**Thoughput**  số lượng dữ liệu chuyển được trỏng một đơn vị thời gian.
	**Throughput** được tính theo công thức:

		Throughput = IOPS * kích thước gói tin trung bình. 
Ví dụ: SSD 330 của Intel có IOPS là 50000. Truyền các gói tin trung bình có kích thước 4KB. Khi đó Throughput = 50000*4=200000KB/s ~200MB/s.

## Một số lệnh thường sử dụng 
### 1. top
Lệnh này để hiển thị các thông tin ram free, cache,  buffers, CPU,... đồng thời hiển thị thông tin các tiến trình như thời gian đã chạy, trạng thái tiến trình, ID tiến trình,...
Các tham số cần nhớ:
	* **PID**			: ID của tiến trình
	* **UID**			: ID của User(Có thể là User của phần mềm hoặc user của người sử dụng)
	* **PPID**		: ID của tiến trình bố mẹ.
	* **USER**		: Tên của người đang sử dụng tiến trình đó.
	* **PRI**			: Độ ưu tiên.
	* **NI**			: Độ ổn định.(Tiến trình đó có thực sự ổn định khi thay đổi độ ưu tiên).
	* **SIZE**		: Lượng bộ nhớ(Code + Data + Stack) được sử dụng tính bằng đơn vị KB
	* **TTY**			: Terminal
	* **RSS**			: Lượng RAM vật lý được sử dụng.
	* **SHARE**		: Lượng bộ nhớ được chia sẻ cùng với các tiến trình khác được tính bằng đơn vị KB 
	* **STAT**		: Trạng thái tiến trình: S= sleeping(Tiến trình vẫn chờ hoạt động). R = Running(Tiến trình đang hoạt động), T=Stop(Tiến trình đã bị dừng), 
					  D = interruptible sleep(Trước khi ngủ có timeout để thức dậy sớm hơn timeout). Z=Zombie(Tiến trình mẹ được hủy nhưng chưa hủy tiến trình con).
					  Khi một tiến trình đang chạy xác định mà ta ấn Ctr+Z: thì tiến trình đó sẽ bị Sleep. Ctrl+C: Tiến trình sẽ Stop 
	* **%CPU**		: Lượng CPU đã sử dụng.
	* **%MEM**		: Lượng RAM đã sử dụng.
	* **TIME**		: Tổng thời gian CPU đã được sử dụng cho tiến trình đó. 
	* **COMMAND**	: Lệnh được sử dụng để bắt đầu các task. 

Khi lệnh **top** đang chạy. Có thể dùng một số phím tắt sau:
	* **t**			: Xem thông tin tóm tắt.
	* **m**			: Hiển thị thông tin về bộ nhớ như RAM và SWAP. Các dạng hiển thị. 
	* **A**			: Sắp xếp và hiển thị những ứng dụng cần nhiều hiệu năng. Ấn "A" nhiều lần để di chuyển qua các bảng. Ấn "A" sau đó chuột trái để thoát khỏi chế độ.
	* **f**			: Hiển thị giải thích các thông số. Ấn Space sẽ in đậm dòng chữ. Mục đích làm thêm trường vào trong **top**.
	* **k**			: Kill tiến trình. 

### 2. ps và pstree

Đây là 2 lệnh hiển thị thông tin tiến trình theo hai dạng danh sách và dạng cây.
Khác với **top** thì 2 lệnh này chỉ hiện thông tin các tiến trình còn **top** thì hiển thị nhiều thông tin hơn như ram, bộ nhớ, bufter, command, time, ..

Lệnh **ps** có một số tham số mở rộng như sau:
	* **-e**			: Giống hệt -A là hiển thị tất cả các tiến trình.
	* **-l**			: Hiển thị tất cả trường thông tin.
	* **-F**			: 
	* **-f**			: Hiển thị rút gọn hơn -l.
	* **-o**			: Hiển thị ra những trường cần hiển thị.
	* **-H**			: Hiển thị dạng phân cấp.
	* **-L**			: Hiển thị tất cả các luồng của tiến trình. Lệnh này sẽ hiển thị thêm **LWP(light weight process)** và **NLWP(number of light weight process)**
	* **-m**			: Hiển thị những luồng trong tiến trình.
	* **--forest**		: Hiển thị kết hợp dạng cây và danh sách.
	* **--sort**		:Sắp xếp theo chiều giảm dần.
	* **-C + <Tên ứng dụng đang dùng>**	: Hiển thị những tiến trình của ứng dụng đó.

_Ví dụ:_
```sh
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head
  PID  PPID CMD                                       %MEM    %CPU
 6334  2007 /opt/google/chrome/chrome -  6.3	 13.4
 1507  1379 /usr/bin/gnome-shell         	   4.8	 11.8
 7204  2007 /opt/google/chrome/chrome -  3.8	  8.6
 2039  1968 /opt/google/chrome/chrome -  3.4	  2.2
 1968     1 /opt/google/chrome/chrome       2.8	  3.9
 5683  2007 /opt/google/chrome/chrome -  2.4	  0.7
 2235  2007 /opt/google/chrome/chrome -  2.0	  1.0
 6657  2007 /opt/google/chrome/chrome -  1.7	  1.8
 2336  2007 /opt/google/chrome/chrome -  1.7	  0.8

```

**Nguồn: https://www.tecmint.com/ps-command-examples-for-linux-process-monitoring/**

### 3. Free
Lệnh hiển thị thông tin ram và swap như ram free, ram available, ...
	* **-m** hiển thị thông tin đạng MB.


### 4. iostat 
Hiển thị thông tin của CPU và I/O của các phân vùng. 
**avg-cpu** là thông tin của CPU với các trường sau:
	* **%user**		: Tỷ lệ phần trăm người dùng sử dụng.
	* **%nice**		: Tỷ lệ phần trăm đã sử dụng của các service có độ ưu tiên cao.
	* **%sys**		: Tỷ lệ phần trăm hệ thống đã sử dụng.
	* **%idle**		: Tỷ lệ phần trăm của CPU chưa sử dụng. 

**Device** là thông tin ổ đĩa tốc độ đọc ghi của hệ thống với các trường sau: 
	* **tps**					: Số transfers chuyển được trên 1s đến thiết bị. Nhiều I/O đơn có thể cùng được gửi trong 1 transfer.
	* **Blk_read/s, Blk_wrtn/s**	: Tốc độ đọc ghi dữ liệu.
	* **Blk_read, Blk_wrtn**		: Tổng số đọc ghi dữ liệu. 

Một số tham số mở rộng hữu ích. 
	* **-p + tên thiết bị CPU hoặc đĩa**			: Hiển thị thông tin của riêng thiết bị chỉ sẵn.
		VD: iostat -p cpu
	* **-N**									: Hiển thị thông tin LVM của ổ đĩa.

###  5. netstat
Hiển thị các thông tin trong mạng như socket sử dụng, routing, interface, protocal, ....
Các tham số thường gặp:
	* **-a**			: Hiển thị tất cả thông tin socket.
	* **-r**			: Thông tin routing như Device, netmark, Interface,...
	* **-i**			: Thông tin thống kê các Interface.
	* **-s**			: Thống kê các giao thức. 
	* **-l**			: Liệt kê tất cả các kết nối có cổng nghe đang hoạt động.

Các trường trong socket:
	* **Proto**			: Giao thức.
	* **Local Address** 	: Địa chỉ nội bộ và Port của socket tương ứng. 
	* **Foreign Address**	: Địa chỉ và Port của Socket đích.
	* **State**			: Trạng thái của Socket.

### 6. tcpdump
Lệnh bắt các gói tin 
	* **-i <Name Interface>**	: là gán interface card mạng
	* **-c <Number Packages>** : là số gói tin cần bắt
	* **-w <Name.pcap>**		: lưu trữ gói tin vào một tập tin để sau này phân tích.
	* **-A**					: phân tích về dạng ASCII
	* **tcp**					: để bắt gói tin tcp
	* **port <Port Number> **	: bắt gói tin ở cổng nào

Ví dụ: 
```sh
$  tcpdump -i wlan0 -c 20 -w tcpPort.pcap tcp port 80
```

### 7. nmon
Là một tool tổng hợp của _top_, _iostat_ . Nó thống kê cá thông tin như I/O disks, networkm process, ...

### 8. nmap
Tool dùng để phân tích mạng, quét bảo mật, tìm port mở, ...
**Quét hostname, IP**:
```sh
$ nmap + <DNS/IP>
```

**Quét nhiều Host**:
```sh
$ nmap + <DNS1/IP1> + <DNS2/IP2> + ... +<DNSn/IPn>
```
Ví dụ: nmap 192.168.0.101 192.168.0.102 192.168.0.103
Với quét nhiều host mà có cùng dải mạng thì ta còn một cách khác:
	nmap 192.168.0.101,102,103
Hoặc:
	nmap 192.168.0.101-103

**Quét Subnet**:
```sh
$ nmap 192.168.0.*
```

**Quét cổng của host**:
 * **Quét 1 cổng:**
```sh
$ nmap -p 80 192.168.1.1
```
 * **Quét nhiều cổng:**
```sh
$ nmap -p 1-100 192.168.1.1
```
Hoặc:
```sh
$ nmap -p 1,2,3,4 192.168.1.1
```
 * **Quét tất cả các host:**
```sh
$ nmap -p- 192.168.1.1
```

Một số tùy chọn khác
	* **-O**		: Quét hệ điều hành máy.
	* **-SP**		: Kiểm tra những host sống trong network.
	* **-F**		: Quét nhanh một host.
	* **--iflist**	: Quét Interface.
	* **-p + <Số hiệu cổng>**: Quét cổng xác định.
	* **-sT**		: Quét két nối TCP.
	* **-sU**		: Quét kết nối UDP.

**Tham khảo: https://www.tecmint.com/nmap-command-examples/**
**Tham khảo: https://opensourceforu.com/2011/02/advanced-nmap-scanning-firewalls/**

### 9. Tracepath
Định tuyến gói tin qua các nút mạng và hiển thị
```sh
$ tracepath <DNS/IP>
```

### 10. strace
Là một lệnh ghi lại nhật kí các hàm được tiến trình gọi. Các hàm này thường là các hàm gửi, nhận, đọc,... trong lập trình socket.
```sh
$ strace -p <PID của tiến trình cần theo dõi>
```





