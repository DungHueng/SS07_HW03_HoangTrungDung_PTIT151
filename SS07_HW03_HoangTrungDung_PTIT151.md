# BÀI 3: Đọc hiểu & Dò lỗi qua Prompt (Tái cấu trúc/Tối ưu mã nguồn cũ - Clean Code)
## 1. Phân tích các lỗi vi phạm Clean Code của mã nguồn ban đầu

### Vi phạm SRP (Single Responsibility Principle)

- ReportGenerator đang thực hiện quá nhiều nhiệm vụ:

* Đọc file
* Parse dữ liệu
* Validate dữ liệu
* Tính tổng doanh thu
* Lọc đơn hàng
* Ghi file báo cáo
* In log ra màn hình

### Đặt tên biến không rõ nghĩa

Ví dụ:
f
out
r
l
t
p
w

#### => Không thể hiện ý nghĩa.

- Nên đổi thành: 

inputFile
outputFile
reader
line
totalAmount
fields
writer

### Không kiểm tra dữ liệu đầu vào

Ví dụ: String[] p = l.split(",");

nhưng không kiểm tra

p.length >= 3

Nếu dòng dữ liệu lỗi sẽ phát sinh
ArrayIndexOutOfBoundsException

### Không xử lý NumberFormatException
double amt = Double.parseDouble(p[1]);

Nếu: ABC

thì toàn bộ chương trình bị dừng.

### Không xử lý IOException riêng

Method: throws Exception

là quá chung chung.

Doanh nghiệp thường xử lý riêng:

- IOException
- NumberFormatException
- IllegalArgumentException

### Không dùng try-with-resources

Hiện tại: 

BufferedReader r = ...
...
r.close();

Nếu exception xảy ra giữa chừng

close() không được gọi.

### Logging sai chuẩn
System.out.println(...)

Không phù hợp Production.

Nên dùng

@Slf4j
log.info()
log.warn()
log.error()

### Không có logging lỗi

Không biết dòng nào lỗi.

Không biết file nào lỗi.

Không biết nguyên nhân.

### Logic lồng nhau quá sâu
while
    if
        if

#### => Khó đọc.

#### => Nên dùng Guard Clause.

### Khó mở rộng

Nếu sau này cần

- CSV
- Excel
- PDF

thì phải sửa trực tiếp class hiện tại

## Chuỗi Prompt Cải tiến đầu ra

#### Vòng 1 – Robustness
Prompt

Bạn là Senior Java Developer.

Hãy refactor đoạn code dưới đây nhưng CHƯA thay đổi kiến trúc.

Yêu cầu:

1. Kiểm tra null và chuỗi rỗng.
2. Kiểm tra số lượng phần tử sau khi split().
3. Xử lý IOException bằng try-with-resources.
4. Xử lý NumberFormatException.
5. Nếu gặp dòng dữ liệu lỗi thì ghi chú và bỏ qua dòng đó, không dừng toàn bộ chương trình.
6. Không dùng throws Exception.
7. Giữ nguyên chức năng hiện tại.
8. Giải thích từng thay đổi đã thực hiện.

#### Vòng 2 – Clean Code & SRP
Prompt

Tiếp tục cải tiến kết quả ở vòng trước.

Yêu cầu:

1. Áp dụng nguyên tắc Single Responsibility Principle.
2. Tách thành các phương thức hoặc lớp riêng:

- FileReaderService
- ReportService
- ReportWriter

3. Đổi toàn bộ tên biến theo CamelCase.
4. Loại bỏ các if lồng nhau bằng Guard Clause.
5. Sử dụng try-with-resources ở mọi thao tác IO.
6. Viết code theo chuẩn Clean Code của doanh nghiệp.
7. Không thay đổi chức năng.

#### Vòng 3 – Logging & Context Tuning
Prompt

Tiếp tục refactor kết quả ở vòng trước.

Yêu cầu:

1. Sử dụng Lombok @Slf4j.
2. Thay thế toàn bộ System.out.println bằng log.info().
3. Khi xảy ra lỗi đọc file hoặc parse dữ liệu, dùng log.warn() hoặc log.error().
4. Không hardcode tham số.
5. Truyền đường dẫn file qua tham số hoặc cấu hình.
6. Viết mã nguồn theo chuẩn Spring Boot Enterprise.
7. Sinh đầy đủ mã nguồn cuối cùng.

## Mã nguồn Java hoàn chỉnh sau tối ưu

    import lombok.Data;
    import lombok.extern.slf4j.Slf4j;
    
    import java.io.*;
    import java.util.ArrayList;
    import java.util.List;
    
    /**
     * Entity lưu thông tin báo cáo.
     */
    @Data
    class ReportResult {
    
        private double totalAmount;
        private List<String> completedOrderIds = new ArrayList<>();
    
    }
    
    /**
     * Đọc file.
     */
    @Slf4j
    class OrderFileReader {
    
        public List<String> readLines(String inputFile) {
    
            List<String> lines = new ArrayList<>();
    
            if (inputFile == null || inputFile.isBlank()) {
                log.error("Input file is null or empty.");
                return lines;
            }
    
            try (BufferedReader reader = new BufferedReader(new FileReader(inputFile))) {
    
                String line;
    
                while ((line = reader.readLine()) != null) {
                    lines.add(line);
                }
    
            } catch (IOException e) {
                log.error("Cannot read file: {}", inputFile, e);
            }
    
            return lines;
        }
    }

 * Xử lý nghiệp vụ.

        @Slf4j
        class ReportService {
        
            public ReportResult process(List<String> lines) {
        
                ReportResult result = new ReportResult();
        
                for (String line : lines) {
        
                    if (line == null || line.isBlank()) {
                        continue;
                    }
        
                    String[] fields = line.split(",");
        
                    if (fields.length < 3) {
                        log.warn("Invalid record skipped: {}", line);
                        continue;
                    }
        
                    if (!"COMPLETED".equals(fields[2])) {
                        continue;
                    }
        
                    try {
        
                        double amount = Double.parseDouble(fields[1]);
        
                        if (amount <= 100) {
                            continue;
                        }
        
                        result.setTotalAmount(result.getTotalAmount() + amount);
                        result.getCompletedOrderIds().add(fields[0]);
        
                    } catch (NumberFormatException ex) {
        
                        log.warn("Invalid amount '{}', skipped record: {}", fields[1], line);
        
                    }
        
                }
        
                return result;
            }
        }

 * Ghi báo cáo.

        @Slf4j
        class ReportWriter {
        
            public void write(String outputFile, ReportResult reportResult) {
        
                if (outputFile == null || outputFile.isBlank()) {
                    log.error("Output file is null or empty.");
                    return;
                }
        
                try (PrintWriter writer = new PrintWriter(new FileWriter(outputFile))) {
        
                    writer.println("Total: " + reportResult.getTotalAmount());
        
                    for (String orderId : reportResult.getCompletedOrderIds()) {
                        writer.println("Order ID: " + orderId);
                    }
        
                    log.info("Report successfully written to {}", outputFile);
        
                } catch (IOException e) {
        
                    log.error("Cannot write report: {}", outputFile, e);
        
                }
            }
        }

 * Điều phối toàn bộ quá trình tạo báo cáo.

        @Slf4j
        public class ReportGenerator {
        
            private final OrderFileReader orderFileReader = new OrderFileReader();
            private final ReportService reportService = new ReportService();
            private final ReportWriter reportWriter = new ReportWriter();
        
            public void generate(String inputFile, String outputFile) {
        
                log.info("Start generating report.");
        
                List<String> lines = orderFileReader.readLines(inputFile);
        
                ReportResult reportResult = reportService.process(lines);
        
                reportWriter.write(outputFile, reportResult);
        
                log.info("Report generation completed.");
            }
        }
