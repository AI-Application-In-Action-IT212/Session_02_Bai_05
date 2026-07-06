# BÀI 5: Sáng tạo (Thiết kế Quy trình & Prompt Kiểm chứng Đầu ra - AI Verifier)

## 1. Bối cảnh và ý đồ thiết kế quy trình

Trong thực tế, mã nguồn do AI sinh ra không phải lúc nào cũng chính xác hoặc an toàn. Vì vậy, thay vì chỉ sử dụng một prompt sinh mã nguồn, quy trình được chia thành hai bước độc lập:

* **Bước 1 (Code Generation):** Yêu cầu AI đóng vai lập trình viên để sinh mã nguồn theo đúng yêu cầu nghiệp vụ.
* **Bước 2 (AI Auditing):** Yêu cầu một AI khác hoặc một phiên làm việc mới đóng vai kỹ sư kiểm thử độc lập để phản biện, tìm lỗi biên, lỗi logic và các rủi ro kỹ thuật.

Quy trình này giúp giảm nguy cơ tin tưởng tuyệt đối vào kết quả đầu tiên của AI và tăng khả năng phát hiện lỗi trước khi đưa vào sử dụng.

---

## 2. Prompt sinh mã nguồn (Bước 1)

```text
Bạn là lập trình viên Java cao cấp.

Hãy viết lớp TicketValidator bằng Java để kiểm tra tính hợp lệ của mã vé xe theo các quy tắc:

1. Mã vé phải có định dạng:
   BUS-XX-YYMMDD

2. BUS- là tiền tố bắt buộc.

3. XX là mã tỉnh/thành gồm đúng 2 ký tự in hoa từ A-Z.

4. YYMMDD là ngày đi xe.

5. Phải kiểm tra:
   - ticketCode không được null.
   - ticketCode không được rỗng hoặc chỉ chứa khoảng trắng.
   - định dạng phải đúng.
   - ngày tháng phải hợp lệ (không chấp nhận ngày 32, tháng 13,...).
   - ngày đi không được nằm trong quá khứ.

6. Trả về true nếu hợp lệ, false nếu không hợp lệ.

7. Viết đầy đủ import và giải thích ngắn gọn cách hoạt động.
```

---

## 3. Prompt kiểm chứng độc lập (Bước 2)

```text
Bạn là một Kỹ sư Kiểm thử phần mềm và Kiểm toán mã nguồn độc lập.

Hãy phân tích đoạn mã Java dưới đây như một người phản biện, không phải người đã viết ra nó.

Nhiệm vụ:

1. Tìm các lỗi logic tiềm ẩn.
2. Tìm lỗi xử lý ngày tháng.
3. Kiểm tra khả năng phát sinh NullPointerException.
4. Kiểm tra tính đúng đắn khi chuyển giao năm mới.
5. Kiểm tra việc xử lý dữ liệu đầu vào bất thường.
6. Kiểm tra khả năng xảy ra lỗi liên quan đến múi giờ hệ thống.
7. Đề xuất giải pháp cải tiến theo Java hiện đại.
8. Nếu phát hiện vấn đề, hãy cung cấp phiên bản mã nguồn đã sửa và giải thích lý do.

Đây là mã nguồn cần kiểm toán:

[PASTE SOURCE CODE HERE]
```

---

## 4. Minh chứng thực tế

### Bước 1 - Prompt gửi AI

```text
Bạn là lập trình viên Java cao cấp.

Hãy viết lớp TicketValidator bằng Java để kiểm tra tính hợp lệ của mã vé xe theo các quy tắc:
BUS-XX-YYMMDD
...
```

### AI sinh mã nguồn lần đầu

```java
import java.text.SimpleDateFormat;
import java.util.Date;

public class TicketValidator {

    public boolean isValid(String ticketCode) {

        if (ticketCode == null || ticketCode.trim().isEmpty()) {
            return false;
        }

        if (!ticketCode.matches("BUS-[A-Z]{2}-\\d{6}")) {
            return false;
        }

        try {

            String datePart = ticketCode.substring(7);

            SimpleDateFormat sdf =
                    new SimpleDateFormat("yyMMdd");

            Date travelDate = sdf.parse(datePart);

            return !travelDate.before(new Date());

        } catch (Exception e) {
            return false;
        }
    }
}
```

### Bước 2 - AI Auditing phản biện

```text
Phát hiện các vấn đề:

1. SimpleDateFormat mặc định ở chế độ lenient=true.
   Ví dụ:
   261332
   có thể được tự động chuyển đổi thành một ngày hợp lệ
   thay vì báo lỗi.

2. So sánh Date với new Date()
   bao gồm cả giờ phút giây.

   Vé đi hôm nay nhưng thời gian 00:00:00
   có thể bị đánh giá là đã nằm trong quá khứ.

3. substring(7) phụ thuộc vào vị trí cố định.
   Nếu thay đổi định dạng có thể gây lỗi.

4. API Date và SimpleDateFormat đã cũ.
   Java hiện đại khuyến nghị sử dụng java.time.

5. Chưa xử lý rõ ràng các tình huống chuyển năm.

Khuyến nghị:
- Sử dụng LocalDate.
- Sử dụng DateTimeFormatter với ResolverStyle.STRICT.
- So sánh bằng LocalDate.now().
```

### Mã nguồn cuối cùng sau kiểm duyệt

```java
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeParseException;
import java.time.format.ResolverStyle;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class TicketValidator {

    private static final Pattern TICKET_PATTERN =
            Pattern.compile("^BUS-([A-Z]{2})-(\\d{6})$");

    private static final DateTimeFormatter FORMATTER =
            DateTimeFormatter.ofPattern("uuMMdd")
                    .withResolverStyle(ResolverStyle.STRICT);

    public static boolean isValid(String ticketCode) {

        if (ticketCode == null ||
                ticketCode.trim().isEmpty()) {
            return false;
        }

        Matcher matcher =
                TICKET_PATTERN.matcher(ticketCode);

        if (!matcher.matches()) {
            return false;
        }

        String datePart = matcher.group(2);

        try {

            LocalDate travelDate =
                    LocalDate.parse(datePart, FORMATTER);

            return !travelDate.isBefore(LocalDate.now());

        } catch (DateTimeParseException e) {
            return false;
        }
    }
}
```

### Kết quả

Sau khi áp dụng quy trình AI Verifier hai bước, mã nguồn cuối cùng đã:

* Kiểm tra null và chuỗi rỗng.
* Kiểm tra đúng định dạng BUS-XX-YYMMDD.
* Chặn ngày không hợp lệ như 261332.
* Sử dụng LocalDate hiện đại thay cho Date.
* So sánh ngày chính xác mà không bị ảnh hưởng bởi giờ phút giây.
* Dễ bảo trì và đáng tin cậy hơn trong môi trường thực tế.

```
```
