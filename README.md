# Phân tích kết quả JMH Benchmark

Bảng kết quả của JMH (Java Microbenchmark Harness) chứa rất nhiều thông tin giá trị. Dưới đây là phân tích chi tiết.

---

### 1. Giải thích ý nghĩa của mỗi cột

Đây là ý nghĩa của các cột trong bảng kết quả của bạn:

*   **`Benchmark`**: Tên của bài benchmark đang được chạy. Nó thường có định dạng `TênLớp.tênPhươngThức`. Các dòng có dấu hai chấm (`:`) theo sau là các *kết quả phụ* (secondary results) từ các profiler được kích hoạt (như GC profiler).
*   **`(algorithm)`**: Đây là một tham số (`@Param`) của benchmark. Cột này cho biết giá trị của tham số `algorithm` cho lần chạy cụ thể đó (ví dụ: `APRIORI`).
*   **`(datasetSize)`**: Một tham số (`@Param`) khác, cho biết kích thước của bộ dữ liệu đầu vào.
*   **`Mode`**: Chế độ benchmark. Giá trị `avgt` là viết tắt của `AverageTime` (Thời gian trung bình). Chế độ này đo lường thời gian trung bình để thực thi một thao tác (operation).
*   **`Cnt`**: Viết tắt của "Count". Đây là số lần lặp lại của giai đoạn đo lường (measurement iterations). Trong trường hợp của bạn, giá trị là `2`, nghĩa là kết quả `Score` được tính trung bình từ 2 lần chạy đo lường.
*   **`Score`**: Đây là kết quả chính của benchmark. Ý nghĩa của nó phụ thuộc vào `Mode` và `Units`. Với `Mode` là `avgt`, đây là giá trị hiệu suất trung bình.
*   **`Error`**: Sai số thống kê của `Score` (thường là khoảng tin cậy 99.9%). Giá trị này cho biết mức độ ổn định của các kết quả. Sai số càng nhỏ, kết quả càng đáng tin cậy. Khi số lần lặp lại (`Cnt`) quá thấp (như 2), JMH có thể không tính được sai số, nên cột này bị bỏ trống.
*   **`Units`**: Đơn vị của `Score` và `Error`. Ví dụ: `ms/op` (mili giây mỗi thao tác), `MB/sec` (Megabyte mỗi giây), `B/op` (Byte mỗi thao tác).

---

### 2. Giải thích ý nghĩa của mỗi dòng Benchmark (cho APRIORI)

Mỗi dòng trong bảng kết quả cung cấp một loại thông tin khác nhau về hiệu suất của thuật toán `APRIORI` với `datasetSize = 100`.

*   **`DataMiningAlgorithmBenchmark.benchmarkAverageExecutionTime`**
    *   **Ý nghĩa:** Đây là kết quả chính (primary result). Nó đo lường **thời gian thực thi trung bình** của một lần gọi phương thức benchmark. Đây là thước đo hiệu suất cốt lõi mà bạn quan tâm nhất.

*   **`...:gc.alloc.rate`**
    *   **Ý nghĩa:** Đây là kết quả từ GC Profiler. Nó đo lường **tốc độ cấp phát bộ nhớ** trên heap trong quá trình benchmark. Nó cho bạn biết ứng dụng của bạn đang tạo ra "rác" nhanh như thế nào.

*   **`...:gc.alloc.rate.norm`**
    *   **Ý nghĩa:** Đây là **tốc độ cấp phát bộ nhớ đã được chuẩn hóa** (normalized). Thay vì đo theo giây, nó đo lường lượng bộ nhớ (tính bằng byte) được cấp phát **cho mỗi thao tác** (`B/op`). Chỉ số này rất hữu ích để so sánh trực tiếp mức tiêu thụ bộ nhớ giữa các thuật toán, bất kể chúng chạy nhanh hay chậm.

*   **`...:gc.count`**
    *   **Ý nghĩa:** Nó đếm **số lần bộ dọn rác (Garbage Collector - GC) được kích hoạt** trong một thao tác.

*   **`...:gc.time`**
    *   **Ý nghĩa:** Nó đo lường **tổng thời gian (tính bằng mili giây) mà ứng dụng phải tạm dừng** để bộ dọn rác hoạt động. Thời gian này càng cao, ứng dụng của bạn càng bị "khựng" nhiều.

*   **`...:stack`**
    *   **Ý nghĩa:** Đây là kết quả từ Stack Profiler. Nó không thể được biểu diễn bằng một con số duy nhất, do đó `Score` là `NaN` (Not a Number). Dữ liệu chi tiết của nó (phân tích các phương thức nào tốn nhiều thời gian nhất) đã được in ra ở phần log phía trên của kết quả.

---

### 3. Giải thích ý nghĩa giá trị của bảng (cho APRIORI)

Bây giờ, hãy áp dụng các định nghĩa trên để đọc các giá trị cụ thể cho thuật toán `APRIORI`:

*   **`Score: 10.938`, `Units: ms/op`**
    *   **Giải thích:** Trung bình, một lần chạy thuật toán APRIORI với 100 bản ghi mất **10.938 mili giây**.

*   **`Score: 99.286`, `Units: MB/sec`**
    *   **Giải thích:** Trong quá trình chạy, bộ nhớ được cấp phát với tốc độ trung bình là **99.286 Megabyte mỗi giây**. Đây là một con số khá cao, chủ yếu do phương thức `simulateMemoryIntensiveWork` gây ra.

*   **`Score: 1138768.009`, `Units: B/op`**
    *   **Giải thích:** Để hoàn thành một lần chạy thuật toán, chương trình đã cấp phát trung bình **1,138,768.009 byte** (khoảng 1.14 MB) bộ nhớ mới. Đây là chỉ số tốt nhất để so sánh mức độ "ngốn" bộ nhớ giữa các thuật toán.

*   **`Score: 1.000`, `Units: counts`**
    *   **Giải thích:** Trung bình, có **1 chu kỳ dọn rác** xảy ra trong mỗi lần chạy thuật toán.

*   **`Score: 1.000`, `Units: ms`**
    *   **Giải thích:** Thời gian trung bình mà chương trình bị tạm dừng để dọn rác trong một lần chạy là **1 mili giây**.

*   **`Score: NaN`, `Units: ---`**
    *   **Giải thích:** Kết quả phân tích stack không phải là một con số, bạn cần xem chi tiết ở phần log bên trên để biết phương thức nào đang chiếm nhiều thời gian xử lý nhất.
