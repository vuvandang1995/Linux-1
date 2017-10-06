# ZFS 

## Cài đặt ZFS 

**Yêu cầu :**
- Định dạng filesystem : ext4

Ubuntu : 
$ sudo apt install zfs

(*)ZFS hỗ trợ Ubuntu 64-bit

- Tạo zpool với các ổ đĩa : sdb, sdc, sdd, sde :
$ sudo zpool create -f  ZFS-demo /dev/sdb /dev/sdc /dev/sdd /dev/sde

+ tùy chọn " -f " sẽ ghi đè lên bất cứ lỗi nào xuất hiện như trên đĩa đã có dữ liệu ..
+ ZFS-demo là tên của pool

- Sau khi đã tạo xong pool, kiểm tra nó bằng lệnh `df` hoặc `sudo zfs list` :
`df -h /ZFS-demo` :

Filesystem      Size  Used Avail Use% Mounted on
ZFS-demo         78G     0   78G   0% /ZFS-demo

+ Như bạn có thể thấy, ZFS-demo đã được tự động mount và sẵn sàng để sử dụng.

- Nếu bạn muốn xem những ổ đĩa nào đang được sử dụng trong pool , dùng lệnh ` sudo zpool status` :
![Imgur](https://i.imgur.com/r2z9fWC.png)

- Chúng ta vừa tạo một pool gồm 4 ổ đĩa cứng với dung lượng là 78G theo hình thức striped pool. 
Trong ZFS, mặc định khi tạo pool , hình thức đọc/ghi dữ liệu sẽ là stripped . 

+ Stripped có nghĩa là mọi dữ liệu đọc/ghi vào mountpoint đều dựa diễn ra đồng thời ở tất cả các ổ
cứng trong pool. Ví dụ ta tạo 1 file 3KB trên /ZFS-demo , 1 kb sẽ lưu trên sdb, 1 kb trên sdc, 1 kb trên sde.
Tốc độ đọc/ghi sẽ nhanh gấp 3 lần với hình thức Linear. 

- Tuy nhiên, khi một ổ trong 3 bị hỏng dữ liệu sẽ bị mất đi tính toàn vẹn và có thể sẽ không sử dụng
được nữa. Để khắc phục điều này, ta có một cơ chế khác : **RAID-Z pool** .

+ Đầu tiên, xóa bỏ zpool mới tạo là ZFS-demo :
(*) Đảm bảo rằng bạn không ở trong thư mục mountpoint và không có tiến trình nào đang sử dụng trong pool.

`$ sudo zpool destroy ZFS-demo`

+  Sử dụng 3 ổ đĩa để tạo một RAID-Z pool .
```
RAID-Z 
	- Một bản nâng cấp từ RAID 5
	- Tránh được "Write hone" bằng cách sử dụng COW - Copy on write ( khi mất điện đột ngột lúc đang ghi dữ liệu, sẽ có trường hợp không thể biết
	được data blocks howacj parity blocks nào vừa được ghi trùng với dữ liệu đã được ghi trong các ổ stripped,
	và không xác định được là dữ liệu nào đã bị lỗi, đó gọi là hiện tượng "write hole" ).
	- Yêu cầu ít nhất 3 ổ đĩa cứng trở lên. 
	=> Vẫn đảm bảo tốc độ đọc/ghi bằng stripe
	=> Khi một trong 3 ổ đĩa hỏng, dữ liệu vẫn được phục hồi. Trừ khi cả 3 ổ cùng hỏng.
	- Nếu muốn thêm an toàn, có thể dùng RAID 6 ( RAID-Z2 trên cơ sở của ZFS) để có 2 lần dự phòng.
```
`$ sudo zpool create -f ZFS-demo raidz /dev/sdb /dev/sdc /dev/sdd`

+ Sử dụng lệnh `df -h /ZFS-demo` để kiểm tra, kết quả như sau :

```
root@ubuntu:~# df -h /ZFS-demo
Filesystem      Size  Used Avail Use% Mounted on
ZFS-demo         39G     0   39G   0% /ZFS-demo
```
Như vậy, `df -h` chỉ ra rằng 78G ban đầu của Pool đã giảm xuống chỉ còn 39G , 37G sẽ được sử dụng để lưu giữ
các thông tin dự phòng ( parity information ).

`zpool status` cho ta thấy đã chuyển sang hình thức RAID-Z :

![Imgur](https://i.imgur.com/VcNrLp8.png)

- Việc thêm bộ nhớ vào ZFS pool rất dễ dàng : 

![Imgur](https://i.imgur.com/O0cQ3Ky.png)
