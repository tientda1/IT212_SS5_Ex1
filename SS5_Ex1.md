# BÀI 1: Phân tích & Lựa chọn (Kỹ thuật Prompt suy luận theo các bước - Chain-of-thought)

**Đáp án lựa chọn:** Phương án B

### 1. Phân tích chi tiết tại sao Phương án B là tối ưu nhất

Phương án B là một ví dụ xuất sắc về việc kết hợp **cấu trúc 5 thành phần** và kỹ thuật **Chain-of-Thought (CoT)** để giải quyết một bài toán logic phức tạp:

* **Về mặt kỹ thuật Chain-of-Thought (Suy luận theo các bước):**
    * Thay vì yêu cầu AI "viết code ngay" (zero-shot) như thông thường, prompt này ép AI phải "chậm lại" và suy luận theo trình tự: *Phân tích công thức -> Liệt kê bậc thuế -> Chạy thử bằng tay (Dry-run) -> Viết code*.
    * Việc ép AI chạy thử với ví dụ "thu nhập 30 triệu, 1 người phụ thuộc" giúp mô hình tự "neo" lại các giá trị trung gian (thu nhập tính thuế là bao nhiêu, rơi vào bậc thuế nào). Nhờ bước nháp này, khi bắt tay vào sinh mã nguồn, thuật toán rẽ nhánh `if-else` cho các bậc thuế lũy tiến sẽ chính xác tuyệt đối, không bị sai lệch ở các ranh giới bậc thuế (boundary conditions).
* **Về cấu trúc 5 thành phần cốt lõi:**
    * **Role (Vai trò):** *"Java Developer chuyên nghiệp kiêm chuyên gia tài chính"*. Định hướng AI sử dụng tư duy tài chính (chú trọng tính chính xác của tiền tệ) kết hợp với kỹ năng lập trình.
    * **Goal (Mục tiêu):** Viết class Java tính thuế TNCN lũy tiến.
    * **Context & Constraints (Ngữ cảnh & Ràng buộc):** Rất chặt chẽ. Cung cấp sẵn các bậc thuế giả định (tránh AI tự "ảo giác" hoặc dùng biểu thuế của quốc gia khác/năm khác). Yêu cầu bắt buộc sử dụng `BigDecimal` thay vì `double` để triệt tiêu hoàn toàn sai số làm tròn số thập phân (floating-point precision) trong các bài toán tài chính.
    * **Format (Định dạng):** Yêu cầu trình bày rõ ràng từng bước phân tích trước khi xuất ra mã nguồn.

### 2. Phân tích lý do loại trừ các phương án còn lại

* **Phương án A:**
    * **Nhược điểm:** Đây là dạng prompt thô (Zero-shot) quá ngắn gọn và thiếu hụt toàn bộ ngữ cảnh, ràng buộc.
    * **Nguy cơ:** AI có thể sử dụng biểu thuế cũ hoặc của nước ngoài do không được cung cấp số liệu cụ thể. AI có xu hướng sử dụng kiểu dữ liệu `double` cơ bản để tính tiền, dẫn đến sai số nghiêm trọng. Ngoài ra, do không có bước suy luận (CoT), AI rất dễ viết sai logic phân bổ tiền vào các bậc thuế lũy tiến.
* **Phương án C:**
    * **Nhược điểm:** Áp dụng công nghệ sai bối cảnh (Over-engineering).
    * **Nguy cơ:** Yêu cầu sử dụng "Java Stream API song song (parallel) để tính toán cực nhanh các bậc thuế" là một sai lầm về mặt khoa học máy tính. Thuế lũy tiến tính trên một cá nhân là bài toán rẽ nhánh tuần tự (số tiền dư ở bậc dưới mới được đem tính tiếp ở bậc trên). Việc ép AI dùng luồng song song cho bài toán rẽ nhánh đơn trị không những không làm code nhanh hơn mà còn gây lỗi logic nặng nề, code khó đọc và phung phí tài nguyên hệ thống quản lý luồng (thread overhead).

### 3. Mã nguồn Java sinh ra từ Phương án B

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

public class PersonalIncomeTaxCalculator {

    // Các khoản giảm trừ (Đơn vị: VND)
    private static final BigDecimal PERSONAL_DEDUCTION = new BigDecimal("11000000");
    private static final BigDecimal DEPENDENT_DEDUCTION = new BigDecimal("4400000");

    // Các mốc thu nhập tính thuế lũy tiến (Đơn vị: VND)
    private static final BigDecimal TIER_1_MAX = new BigDecimal("5000000");
    private static final BigDecimal TIER_2_MAX = new BigDecimal("10000000");
    private static final BigDecimal TIER_3_MAX = new BigDecimal("18000000");
    private static final BigDecimal TIER_4_MAX = new BigDecimal("32000000");
    private static final BigDecimal TIER_5_MAX = new BigDecimal("52000000");
    private static final BigDecimal TIER_6_MAX = new BigDecimal("80000000");

    // Thuế suất tương ứng
    private static final BigDecimal RATE_1 = new BigDecimal("0.05");
    private static final BigDecimal RATE_2 = new BigDecimal("0.10");
    private static final BigDecimal RATE_3 = new BigDecimal("0.15");
    private static final BigDecimal RATE_4 = new BigDecimal("0.20");
    private static final BigDecimal RATE_5 = new BigDecimal("0.25");
    private static final BigDecimal RATE_6 = new BigDecimal("0.30");
    private static final BigDecimal RATE_7 = new BigDecimal("0.35");

    /**
     * Tính tổng thuế TNCN phải nộp
     *
     * @param totalIncome        Tổng thu nhập
     * @param numberOfDependents Số người phụ thuộc
     * @param hasInsurance       Có đóng bảo hiểm (10.5%) hay không
     * @return Số tiền thuế phải nộp
     */
    public BigDecimal calculateTax(BigDecimal totalIncome, int numberOfDependents, boolean hasInsurance) {
        BigDecimal insuranceDeduction = BigDecimal.ZERO;
        
        // Tính bảo hiểm 10.5% nếu có
        if (hasInsurance) {
            insuranceDeduction = totalIncome.multiply(new BigDecimal("0.105"));
        }

        // Tổng giảm trừ gia cảnh
        BigDecimal totalDependentDeduction = DEPENDENT_DEDUCTION.multiply(new BigDecimal(numberOfDependents));
        BigDecimal totalDeductions = PERSONAL_DEDUCTION.add(totalDependentDeduction).add(insuranceDeduction);

        // Thu nhập tính thuế (TNTT) = Tổng thu nhập - Các khoản giảm trừ
        BigDecimal taxableIncome = totalIncome.subtract(totalDeductions);

        // Nếu TNTT <= 0 thì không phải nộp thuế
        if (taxableIncome.compareTo(BigDecimal.ZERO) <= 0) {
            return BigDecimal.ZERO;
        }

        return calculateProgressiveTax(taxableIncome).setScale(0, RoundingMode.HALF_UP);
    }

    /**
     * Thuật toán tính thuế lũy tiến từng phần
     */
    private BigDecimal calculateProgressiveTax(BigDecimal taxableIncome) {
        BigDecimal totalTax = BigDecimal.ZERO;
        BigDecimal remainingIncome = taxableIncome;

        // Bậc 1: Đến 5 triệu (5%)
        if (remainingIncome.compareTo(BigDecimal.ZERO) > 0) {
            BigDecimal incomeInThisTier = remainingIncome.min(TIER_1_MAX);
            totalTax = totalTax.add(incomeInThisTier.multiply(RATE_1));
            remainingIncome = remainingIncome.subtract(incomeInThisTier);
        }

        // Bậc 2: 5M - 10M (10%) -> Khoảng cách: 5M
        if (remainingIncome.compareTo(BigDecimal.ZERO) > 0) {
            BigDecimal tierGap = TIER_2_MAX.subtract(TIER_1_MAX);
            BigDecimal incomeInThisTier = remainingIncome.min(tierGap);
            totalTax = totalTax.add(incomeInThisTier.multiply(RATE_2));
            remainingIncome = remainingIncome.subtract(incomeInThisTier);
        }

        // Bậc 3: 10M - 18M (15%) -> Khoảng cách: 8M
        if (remainingIncome.compareTo(BigDecimal.ZERO) > 0) {
            BigDecimal tierGap = TIER_3_MAX.subtract(TIER_2_MAX);
            BigDecimal incomeInThisTier = remainingIncome.min(tierGap);
            totalTax = totalTax.add(incomeInThisTier.multiply(RATE_3));
            remainingIncome = remainingIncome.subtract(incomeInThisTier);
        }

        // Bậc 4: 18M - 32M (20%) -> Khoảng cách: 14M
        if (remainingIncome.compareTo(BigDecimal.ZERO) > 0) {
            BigDecimal tierGap = TIER_4_MAX.subtract(TIER_3_MAX);
            BigDecimal incomeInThisTier = remainingIncome.min(tierGap);
            totalTax = totalTax.add(incomeInThisTier.multiply(RATE_4));
            remainingIncome = remainingIncome.subtract(incomeInThisTier);
        }

        // Bậc 5: 32M - 52M (25%) -> Khoảng cách: 20M
        if (remainingIncome.compareTo(BigDecimal.ZERO) > 0) {
            BigDecimal tierGap = TIER_5_MAX.subtract(TIER_4_MAX);
            BigDecimal incomeInThisTier = remainingIncome.min(tierGap);
            totalTax = totalTax.add(incomeInThisTier.multiply(RATE_5));
            remainingIncome = remainingIncome.subtract(incomeInThisTier);
        }

        // Bậc 6: 52M - 80M (30%) -> Khoảng cách: 28M
        if (remainingIncome.compareTo(BigDecimal.ZERO) > 0) {
            BigDecimal tierGap = TIER_6_MAX.subtract(TIER_5_MAX);
            BigDecimal incomeInThisTier = remainingIncome.min(tierGap);
            totalTax = totalTax.add(incomeInThisTier.multiply(RATE_6));
            remainingIncome = remainingIncome.subtract(incomeInThisTier);
        }

        // Bậc 7: Trên 80M (35%)
        if (remainingIncome.compareTo(BigDecimal.ZERO) > 0) {
            totalTax = totalTax.add(remainingIncome.multiply(RATE_7));
        }

        return totalTax;
    }

    public static void main(String[] args) {
        PersonalIncomeTaxCalculator calculator = new PersonalIncomeTaxCalculator();
        
        // Ví dụ Dry-run: Tổng thu nhập 30M, 1 người phụ thuộc, không bảo hiểm
        BigDecimal totalIncome = new BigDecimal("30000000");
        int dependents = 1;
        boolean hasInsurance = false;

        BigDecimal taxToPay = calculator.calculateTax(totalIncome, dependents, hasInsurance);
        
        System.out.println("Tổng thu nhập: " + totalIncome + " VND");
        System.out.println("Thuế TNCN phải nộp: " + taxToPay + " VND");
        // Kết quả mong đợi theo phân tích tay: 1,440,000 VND
    }
}
```