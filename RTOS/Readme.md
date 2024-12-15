#### TASK
- Task là một tập hợp các câu lệnh, được lưu trong bộ nhớ vi điều khiển, để thực hiện một nhiệm vụ nào đó.
- Còn đối với RTOS, các task này sẽ cần được thực hiện gần như "song song". Vì vậy, mỗi task cần có một "chương trình riêng", ở đây là một function để thực hiện chức năng của chúng. Việc thực hiện đa tác vụ trên cùng một chương trình vi điều khiển (ở đây là 1 core) gọi là Multi-Task.
#### Cơ chế lập lịch - Scheduling
- Lập lịch - Scheduling là một thuật toán để xác định Task nào được thực thi. Về cơ bản, một task sẽ có 4 trạng thái chính:
    - READY: Sẵn sàng chạy.
    - RUNNING: Đang chạy.
    - BLOCKED: Chờ một sự kiện hoặc tài nguyên.
    - SUSPENDED: Bị tạm dừng, không tham gia vào lập lịch.

![1](/RTOS/1.png)
#### Các thuật toán lập lịch
- Kernel - hay gọi là Nhân của hệ điều hành, thực chất là một quy ước có nhiệm vụ điều phối các công việc của RTOS (Quy ước vì thực tế nó vẫn là do con người code nên và nạp vào bộ nhớ, chứ không phải nhúng từ bên ngoài vào bên trong vi điều khiển). Việc lập trình ra các Kernel hay toàn bộ hoạt động của một OS trong vi điều khiển thường được thực hiện bằng ngôn ngữ Assembly, để đảm bảo tốc độ xử lý nhanh nhất của Kernel. 
![2](/RTOS/2.png)
- Round-Robin: mỗi task sẽ có một thời gian thực thi nhất định, gọi là Time Slice, hết khoảng thời gian này thì sẽ phải nhường CPU cho task khác thực hiện. 
    - Nhược điểm: Nếu có 1 task khác cần thời gian chạy ngay, mà trong lúc đó có 2 task khác chạy trước, thì task đó phải đợi 2 Time Slice mới đến lượt nó chạy.
- Preemptive: mong muốn những Task quan trọng sẽ được thực hiện trước, bằng cách gán cho chúng những quyền ưu tiên. Và task nào có quyền ưu tiên cao hơn, sẽ có thể chiếm quyền sử dụng CPU của task đang thực hiện khi cần.
- Cooperative: các task sẽ đợi task khác thực hiện xong thì mới đến lượt.
    - Nhược điểm: Nếu task khác có quyền ưu tiên cao hơn nhưng không thể chiếm quyền điều khiển của task đang chạy.
#### Phần cứng được sử dụng
##### Mỗi Task được chia làm 2 phần:
- TCB(Task Control Block): dùng để điều khiển 1 task, nhiệm vụ chính là lưu trữ lại các ngữ cảnh đang thực hiện của một task, trước khi chuyển qua task khác. 
- Stack: Mỗi task chạy thì đều cần có vùng nhớ dữ liệu để thực thi, ở đây là vùng stack riêng của từng task, khác với vùng nhớ stack của chương trình.   
    - Mỗi task hoạt động như một chương trình bình thường, và có cả vùng stack riêng, với stack thì chúng ta cần bố trí một con trỏ ngăn xếp - Stack Pointer (SP). Ở đây chúng ta sẽ dùng PSP - Process Stack Pointer cho hoạt động của các Task, và MSP - Main Stack Pointer vẫn được sử dụng trong chương trình chính (main).
    ![4](/RTOS/3.png)

##### Systick Timer dùng để đếm thời gian Time-Slice cho việc lập lịch.
##### Hai Exception là PendSV và SVC sử dụng để chuyển đổi giữa các task.

#### Phần mềm
- Nội dung của các Task.
- Thuật toán lập lịch.
- Context Switch - Chuyển đổi ngữ cảnh.
- vTaskDelay.
- Một số cơ chế giao tiếp giữa các Task.

#### SYSTICK TIMER
##### Systick Timer là gì ?
- Systick Timer đúng như tên gọi của nó, là một Timer, nhưng không giống như các ngoại vi Timer, Systick Timer là một Timer độc lập nằm trong lõi Cortex.
- Một điểm khác biệt nữa so với Timer Peripheral, Systick Timer là một Timer 24-bit, đếm ngược, xảy ra ngắt khi giá trị đếm bằng 0, và tự nạp lại giá trị đếm ban đầu. 
##### Hoạt động của Systick Timer
![4](/RTOS/4.png)

- Một Bộ đếm Counter để đếm xuống.
- Một thanh ghi chứa giá trị nạp lại.
- Một cờ báo tràn Timer.
- Một bộ phận để cấu hình xung clock đầu vào.
##### Các thanh ghi Systick
![5](/RTOS/5.png)

- Thanh ghi SYST_CSR - Systick Control & Status Register
![6](/RTOS/6.png)
    - bit 16 là cờ này được tự động set lên 1 khi bộ đếm tràn về 0
    - bit 2 Chọn nguồn cấp xung clock cho timer
        - 0 = External Clock = AHB / 8.
        - 1 = Processor Clock = AHB.
    - bit 1 cho phép ngắt Systick nếu bit này bằng 1.
    - bit 0 cho phép bộ đếm hoạt động.
Khi được ENABLE, bộ đệm sẽ lấy giá trị đã cài đặt sẵn trong thanh ghi Reload và bắt đầu đếm ngược.

- Thanh ghi SYST_RVR - Systick Reload Value Register
![7](/RTOS/7.png)
Thanh ghi này chưa giá trị nạp lại để bắt đầu đếm, nó chỉ sử dụng 24 bit thấp, với rải giá trị từ 0x0000.0001 - 0x00FF.FFFF

- Thanh ghi SYST_CVR - Systick Current Value Register
![8](/RTOS/8.png)
Tương tự Timer Peripheral, Systick Timer cũng có một thanh ghi đếm, thanh ghi này sẽ bắt đầu với giá trị lấy từ thanh ghi SYST_RVR, và đếm ngược về 0. Tương tự như RVR, thanh ghi SYST_CVR chỉ sử dụng 24 bit thấp.

- Thanh ghi SYST_CALIB - Systick Calibration Value Register
![9](/RTOS/9.png)
Thanh ghi này dùng để hiệu chuẩn cho Systick.

```
#define TICK_HZ				1000U
#define HSI_CLOCK			8000000U
#define SYSTICK_TIM_CLK		        HSI_CLOCK

void init_systick_timer(uint32_t tick_hz)
{
	uint32_t *pSRVR = (uint32_t *)0xE000E014;
    //Ép kiểu địa chỉ 0xE000E014 thành 1 con trỏ kiểu uint32_t để con trỏ *pSRVR truy cập đến địa chỉ đó.
	uint32_t *pSCSR = (uint32_t *)0xE000E010;

	//reload value
	uint32_t count_value = (SYSTICK_TIM_CLK/ TICK_HZ) -1;

	//xóa SVR 24bit
	*pSRVR &=~ (0x00FFFFFF);
	//ghi value vào SVR
	*pSRVR |= count_value;

	//enable systick exception request
	*pSCSR |= (1<<1);
	//Clock source
	*pSCSR |= (1<<2);
	//enable counter
	*pSCSR |= (1<<0);
}
```
#### Context Switch
- CPU đang ở trong Task 1, sau đó xảy ra 1 ngắt.
- Ngắt Systick xảy ra, Systick Handler chạy, khi này trong Systick handler mình nên lưu trạng thái của Task 1 vào Stack private của Task 1 bằng cách sử dụng PUSH.
- Sau đó mình sẽ lưu PSP của Task 1, giá trị Stack Pointer mà mình phải lưu
- Sau đó mình lấy giá trị hiện tại của PSP của Task 2 từ Stack Pointer private của Task 2, thông qua lệnh POP.
- Sau đó mình thoát khỏi Exception Handler để chạy Task 2.
