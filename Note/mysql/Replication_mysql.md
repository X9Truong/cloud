### Khái niệm Replication
- MySQL Replication là một quá trình cho phép bạn dễ dàng duy trì nhiều bản sao của dữ liệu MySQL bằng cách cho họ sao chép tự động từ một master tạo ra một cơ sở dữ liệu slave. Điều này rất hữu ích vì nhiều lý do bao gồm việc tạo điều kiện cho sao lưu cho dữ liệu, một cách để phân tích nó mà không sử dụng các cơ sở dữ liệu chính, hoặc chỉ đơn giản là một phương tiện để mở rộng ra.

- Replication mặc định là không đồng bộ, slave không cần phải kết nối vĩnh viễn để nhận được cập nhật từ master. Tùy thuộc vào cấu hình, bạn có thể sao chép tất cả các cơ sở dữ liệu, cơ sở dữ liệu đã chọn, hoặc thậm chí bảng được lựa chọn trong một cơ sở dữ liệu. Thật vậy, Replication có ý nghĩa là “nhân bản”, là có một phiên bản giống hệt phiên bản đang tồn tại, đang sử dụng. Với một cơ sở dữ liệu có nhu cầu lưu trữ lớn, thì đòi hỏi cơ sở dữ liệu phải toàn vẹn, không bị mất mát trước những sự cố ngoài dự đoán là rất cao. Vì vậy, người ta nghĩ ra khái niệm (slave) “nhân bản”, tạo một phiên bản cơ sở dữ liệu giống hệt cơ sở dữ liệu đang tồn tại, và lưu trữ ở một nơi khác, đề phòng có sự cố.

- Server master lưu trữ phiên bản cơ sở dữ liệu phục vụ ứng dụng. Server slave lưu trữ phiên bản cơ sở dữ liệu “nhân bản”. Quá trình nhân bản từ master sang slave gọi là replication.

- Tất cả các thay đổi trên cơ sở dữ liệu master sẽ được ghi lại dưới dạng file log binary, slave đọc file log đó, thực hiện những thao tác trong file log, việc ghi, đọc và thực thi trong file log này dưới dạng binary được thực hiện rất nhanh.

- Ưu điểm của replication trong mysql
* Giảm tải cho cơ sở dữ liệu server master, tải trọng của server được phân tải cho các con slave, cải thiện hiệu năng cho toàn hệ thống. Trong môi trường này, tất cả các quá trình ghi và cập nhật đều phải diễn ra trên server master, bên cạnh đó quá trình đọc được diễn ra trên một hoặc nhiều con slave. Chính vì vậy mô hình này giúp tăng đáng kể hiệu năng của toàn hệ thống.
* Tính bảo mật dữ liệu cao - vì dữ liệu được sao chép đến các slave, và các slave có thể tạm dừng quá trình sao chép, nó có thể chạy các dịch vụ sao lưu trên các slave mà không làm hư hỏng dữ liệu tổng thể tương ứng.
* Tính phân tích - dữ liệu trực tiếp có thể được tạo ra trên master, trong khi phân tích các thông tin có thể xảy ra trên các slave mà không ảnh hưởng đến hiệu suất của master.
* Tính phân phối dữ liệu từ xa - bạn có thể sử dụng replication để tạo ra một bản sao của dữ liệu cho một trang web từ xa để sử dụng, mà không cần truy cập thường xuyên vào con master.


- Replication dựa trên các con master lưu giữ theo dõi tất cả những thay đổi cơ sở dữ liệu của nó (cập nhật, xóa, vv) trong bản ghi nhị phân của nó. Các bản ghi nhị phân phục vụ như là các record của tất cả các sự kiện làm thay đổi cấu trúc cơ sở dữ liệu hoặc nội dung (dữ liệu) từ thời điểm các máy chủ đã bắt đầu thực thi. Thông thường, câu SELECT không được ghi lại bởi vì chúng không phải thay đổi cấu trúc cũng như nội dung của cơ sở dữ liệu.

- Mỗi slave kết nối đến các master yêu cầu một bản sao của bản ghi nhị phân. Đó là, nó kéo các dữ liệu từ các master, chứ không phải là master đẩy dữ liệu đến các slave. Các slave cũng thực hiện các sự kiện từ các bản ghi nhị phân mà nó nhận được. Quá trình này lặp đi lặp lại những thay đổi ban đầu cũng giống như nó đã được thực hiện trên master. Bảng được tạo ra hoặc cấu trúc thay đổi và dữ liệu đã chèn hay đã xóa và kể cả cập nhật thì đều giống hệt theo những thay đổi mà ban đầu đã được thực hiện trên master.

* Chi tiết quá trình thực thi trong Replication như sau:

- Luồng Binlog dump: Các master tạo 1 luồng và gửi nội dung binary log đến một slave khi các slave kết nối với master. Luồng này có thể được xác định trong đầu ra của query SHOW PROCESSLIST trên master như là 1 luồng Binlog Dump. Các binary log có được trên bản ghi nhị phân của master đọc các sự kiện đó và gửi đi cho slave. Ngay sau khi sự kiện được đọc, khóa được phát hành ngay cả khi sự kiện được gửi tới slave.
- Luồng Slave I/O: Khi thông báo Slave được ban hành trên slave server, các slave tạo một luồng I/O, cái mà kết nối với server master và hỏi nó để nó gửi thông tin bản ghi cập nhật nó vào trong log nhị phân. Luồng slave I/O đọc sự cập nhật trên luồng Binlog Dump của master gửi và sao chép chúng vào 1 file local - file mà bao hàm cả những log trễ (Relay Log). Các trạng thái của luồng này được thể hiện như là Slave_IO_running trong output SHOW SLAVE STATUS hoặc là Slave_running trong ouput của SHOW STATUS.
- Luồng Slave SQL: Các slave tạo ra một luồng SQL để đọc cái log trễ, cái này sau đó sẽ được ghi vào luồng slave I/O và thực thi các sự kiện chứa trong đó.

- MySQL Replication là một giải pháp scale out (tăng số lượng instance MySQL) nhưng không phải bài toán nào cũng dùng được. Các bài toán mà MySQL Replication sẽ giải quyết tốt:

* Scale Read
* Data Report
* Real time backup

### 1.1 Scale Read

- Scale Read thường gặp ở các ứng dụng mà số truy vấn đọc dữ liệu nhiều hơn ghi, tỉ lệ read/write có thể 80/20 hoặc hơn. Các ứng dụng thường gặp là báo, trang tin tức.
- 
- Với scale read ta sẽ chỉ có một Master instance phục vụ cho việc đọc/ghi dữ liệu. Có thể có một hoặc nhiều Slave instance chỉ phục vụ cho việc đọc dữ liệu.
- 
- Một số ứng dụng write nhiều (thương mại điện tử) cũng có sử dụng MySQL Replication để scale out hệ thống.

### 1.2 Data Report

Một số hệ thống cho phép một số người (leader, manager, người làm report, thống kê, data) truy cập vào dữ liệu trên production phục vụ cho công việc của họ. Việc chọc thẳng vào data production sẽ rất nguy hiểm vì:

- Vô tình chỉnh sửa làm sai lệnh dữ liệu (nếu có quyền insert, update).
- Vô tình thực thi các câu truy vấn tốn nhiều tài nguyên, thời gian truy vấn dài làm treo hệ thống.
- Việc setup một máy chủ làm data report (application cũng sẽ không kết nối tới server này) làm giảm thiểu 2 rủi ro trên.

### 1.3 Real time backup

- Với cơ sở dữ liệu lớn việc backup không thể thực hiện thường xuyên được (hàng giờ, hàng phút). Với các ứng dụng giao dịch tài chính, thanh toán, TMDT nếu bị mất dữ liệu 1 giờ, 1 ngày thì thiệt hại sẽ rất lớn (máy chủ chính tư dưng bị hỏng). Real time backup là một giải pháp bổ sung cho offline backup, chạy đồng thời cả 2 phương pháp này để bảo đảm sự an toàn cho dữ liệu.


### 2.2 Cách hoạt động

<img src="/Note/img/mysql.jpg">

* Trên Master:

- Các kết nối từ web app tới Master DB sẽ mở một Session_Thread khi có nhu cầu ghi dữ liệu. Session_Thread sẽ ghi các statement SQL vào một file binlog (ví dụ với format của binlog là statement-based hoặc mix). Binlog được lưu trữ trong data_dir (cấu hình my.cnf) và có thể được cấu hình các thông số như kích thước tối đa bao nhiêu, lưu lại trên server bao nhiêu ngày.
- Master DB sẽ mở một Dump_Thread và gửi binlog tới cho I/O_Thread mỗi khi I/O_Thread từ Slave DB yêu cầu dữ liệu.
* Trên Slave:

- Trên mỗi Slave DB sẽ mở một I/O_Thread kết nối tới Master DB thông qua network, giao thức TCP (với MySQL 5.5 replication chỉ hỗ trợ Single_Thread nên mỗi Slave DB sẽ chỉ mở duy nhất một kết nối tới Master DB, các phiên bản sau 5.6, 5.7 hỗ trợ mở đồng thời nhiều kết nối hơn) để yêu cầu binlog.
- Sau khi Dump_Thread gửi binlog tới I/O_Thead, I/O_Thread sẽ có nhiệm vụ đọc binlog này và ghi vào relaylog.
- Đồng thời trên Slave sẽ mở một SQL_Thread, SQL_Thread có nhiệm vụ đọc các event từ relaylog và apply các event đó vào Slave => quá trình replication hoàn thành.


- Về logic mỗi Slave DB sẽ chỉ nhận dữ liệu từ Master DB, mọi hành động cập nhật dữ liệu BẮT BUỘC phải được thực hiện trên Master. Về nguyên tắc nếu ghi dữ liệu trực tiếp lên Slave DB => hỏng replication. Nhưng thực chất ta hoàn toàn có thể ghi dữ liệu trên Slave miễn sao khi Slave đọc binlog và apply không đụng gì tới những trường dữ liệu mà ta mới ghi vào thì sẽ không bị lỗi (mục này sẽ nói thêm ở các phần sau)
- 
- Với MySQL 5.5 thì mỗi slave sẽ chỉ có một slave_thread connect tới Master, tuy nhiên từ phiên bản 5.6 chúng ta có thể cấu hình nhiều slave_thread để việc apply bin log tới các slave nhanh hơn.


### Replication Lag

* Replication Lag là độ trễ dữ liệu của Slave so với Master. Khi triển khai một hệ thống MySQL Replication thì Lag là vấn đề chắc chắn gặp phải. Ta chỉ có thể giảm thiểu độ trễ dữ liệu trong mức chấp nhận được chứ không thể không có lag. Lí do là việc đồng bộ dữ liệu là Asynchronous, nghĩa là các Slave server không cần thông báo cho Master biết khi transaction thực hiện trên Slave thành công -> điều này giúp giữ nguyên hiệu suất (khác với cơ chế đồng bộ synchronous, một transaction được gọi là thành công khi nó committed trên master server và master server nhận được một thông báo từ slave server là transaction này đã được write và committed. Quá trình này đảm bảo tính thống nhất giữa master và slave server nhưng đồng thời nó làm giảm hiệu suất đi một nữa do các vấn đề về network, bandwith, location.)

* Vấn đề của replication lag ảnh hưởng tới các truy vấn vừa write dữ liệu xuống là read dữ liệu trả về ngay lập tức.

### Semi-synchronous

- Có một vấn đề với asynchronous đó là nếu bạn có nhu cầu đọc ngay dữ liệu vừa ghi xuống thì có thể dữ liệu sẽ sai, do slave chưa kịp apply dữ liệu từ master (lag dữ liệu), có 2 cách giải quyết tạm:

- Với trường hợp vừa ghi và đọc liền dữ liệu, ta nên dùng ở master.
- Dùng cơ chế semi-synchronous để giảm lag dữ liệu.
- Semi-synchronous là một kiểu lai giữa asynchronous và synchronous. Bình thường nếu xài synchronous thì càng nhiều slave thì càng giảm tốc độ ghi dữ liệu, do slave phải committed và trả lời ngược về master. Với semi-synchronous thì master coi như ghi thành công là khi có tối thiểu một slave đã nhận và ghi ra relay log event mà master gửi qua. Điểm khác biệt là không cần tất cả các slave gửi tín hiệu ngược lại master, và event cũng không bắt buộc phải được execute và commited trên slave, chỉ cần đảm bảo là đã nhận và ghi ra relay log là đủ.

Như mô tả ở trên thì slave vẫn có thể không có dữ liệu nếu relay log bị tác động với con người, hoặc server bị hỏng ngay khi chưa kịp apply relay log, tuy nhiên nhờ việc đảm bảo binlog event đã đc nhận với slave và ghi xuống disk đã làm giảm thời gian delay và vấn đề về data race condition có thể được hạn chế phần nào.

