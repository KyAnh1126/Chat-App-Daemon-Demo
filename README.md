- Source files giống như chat-app-socket repo, nhưng có thêm đoạn code daemonize để tách biệt tiến trình của server với terminal và giúp tiến trình độc lập hơn
- Tracepoint được dùng để theo dõi trạng thái chương trình hoặc giá trị của các biến tùy vào người dùng trong các ứng dụng thời gian thực như chat app
- Cần có gdbserver để kết nối từ gdbclient nếu chương trình đích không chạy cùng một máy với máy test

Các bước để debug dùng tracepoint trong GNU & ví dụ thực tế:
1. Compile & chạy ứng dụng mong muốn:
    - Ví dụ:
        + g++ -g server.cpp -o server
        + sudo ./server
2. Tìm process ID của tiến trình:
   - Ví dụ: ID của tiến trình server là 8421
   ![test](https://github.com/user-attachments/assets/ade2e99f-01a1-447e-b987-cb3e93da04be)


3. Thiết lập gdbserver để lắng nghe trên cổng mong muốn:
    - Ví dụ: sudo gdbserver --multi --once localhost:55555
   ![image](https://github.com/user-attachments/assets/92bfb26a-8b42-4dd9-95fa-1c7b3a382655)

4. Thiết lập kết nối từ gdbclient và debug lỗi dùng tracepoint:
   - Thiết lập kết nối & gắn tiến trình:
        + sudo gdb -q : khởi chạy gdb với chế độ quiet mode
        + set target-async on : bật chế độ bất đồng bộ của mục tiêu
        + set non-stop on : cho phép dừng 1 luồng và không ảnh hưởng đến các luồng khác
        + set pagination off : tắt tính năng phân trang
        + set remote interrupt-on-connect off : tiến trình đang chạy không bị gián đoạn khi gdb kết nối tới gdbserver từ xa (remote)
        + file /home/kyanh/Desktop/chat_app_tcp_daemon/chat_app_socket/server :
        + target extended-remote localhost:55555 : kết nối đến gdbserver đang lắng nghe tại local
        + attach 8421 & : attach vào tiến trình 8421 là ID của server

    - Dùng các lệnh để debug, hiển thị thông tin read_ret cũng như buffer khi client gửi tin nhắn đến server:
        + info threads hiển thị thông tin các luồng đang chạy
          ![image](https://github.com/user-attachments/assets/4a8fa928-6b60-4bc7-993d-a3817dd8968a)

        + trace server.cpp:106 để đặt tracepoint tại line number 106 & actions 1 (với 1 là id của tracepoint) để đặt action tương ứng là 'collect read_ret' để đọc giá trị read_ret
          ![image](https://github.com/user-attachments/assets/026a3f3d-ac3d-4412-828b-3b8a4ab9c6df)
          
          ![image](https://github.com/user-attachments/assets/cc09271a-ee30-4ec1-8740-ba493fbf3b1b)


        + tstart để khởi chạy debug, thu thập các frame để hiển thị thông tin các biến
        + tstatus hiển thị trạng thái đang debug như thế nào, số frame thu thập được,...
        + tstop dừng debug
        + tfind x để tìm frame x tương ứng, kết hợp với tdump để thực hiện actions được thực hiện với frame x
          ![image](https://github.com/user-attachments/assets/85d91db9-60d6-4caf-b06e-c8383e8307d7)
