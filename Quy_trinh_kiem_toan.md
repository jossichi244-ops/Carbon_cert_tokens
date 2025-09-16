### 🟠 **1. Input ban đầu từ Người dùng / IoT Device**

*   **Input:**
    *   Dữ liệu thô từ cảm biến IoT (mực nước, độ ẩm đất, hình ảnh vệ tinh...).
    *   Hóa đơn mua phân bón hữu cơ, ảnh chụp hiện trường.
    *   Giao dịch bán nông sản hoặc bán token carbon.
*   **Output:**
    *   Bản ghi trong `collection_iot_data_ingestion` (trạng thái `raw`).
    *   Bản ghi trong `collection_tax_documents` (loại `sale_agri` hoặc `sale_token`).

---

### 🟠 **2. Xử lý dữ liệu & Tạo minh chứng**

*   **Input:** Bản ghi `raw` từ `collection_iot_data_ingestion`.
*   **Output:**
    *   Bản ghi minh chứng trong `collection_carbon_evidence` (trạng thái `pending` → sau đó `verified` bởi tổ chức kiểm toán độc lập).
    *   Giá trị `measured_value` (VD: kg CH₄ giảm) được tính toán dựa trên phương pháp luận (methodology).

---

### 🟠 **3. Tạo & Gửi hồ sơ cho Cơ quan Nhà nước**

*   **Input:** Danh sách các minh chứng (`evidence`) đã được `verified`.
*   **Output:**
    *   Bản ghi hồ sơ trong `collection_government_review` (trạng thái `pending_submission` → `submitted` → `approved`).
    *   Hồ sơ bao gồm: `evidence_list`, `calculated_credits`, `supporting_documents`.

---

### 🟠 **4. Đúc Token & NFT sau khi được phê duyệt**

*   **Input:** Hồ sơ trong `collection_government_review` có `review_status = 'approved'`.
*   **Output:**
    *   **NFT:** Được mint trên blockchain, đại diện cho chứng chỉ tín chỉ carbon gốc → Lưu trong `collection_carbon_token_minting`.
    *   **Token ERC-20 (VNCR):** Được tạo ra để giao dịch → Lưu trong `collection_carbon_fungible_tokens`.
    *   **Cập nhật ví:** Số dư token của người dùng được cộng thêm trong `collection_carbon_wallet_balances`.

---

### 🟠 **5. Giao dịch trên Sàn & Tạo chứng từ thuế**

*   **Input:**
    *   Người dùng đặt lệnh mua/bán token → `collection_carbon_orders`.
    *   Người dùng bán nông sản → `collection_tax_documents`.
*   **Output:**
    *   Giao dịch được khớp → Lưu trong `collection_carbon_transactions`.
    *   Số dư ví được cập nhật → `collection_carbon_wallet_balances`.
    *   Chứng từ thuế tự động được tạo → `collection_tax_documents`.

---

### 🟠 **6. Kiểm toán phi tập trung (Decentralized Audit)**

Đây là trái tim của quy trình kiểm toán, đảm bảo tính minh bạch và không thể thay đổi.

*   **Bước 1: Nông dân ký tờ khai**
    *   **Input:** Tờ khai thuế tháng (draft) trong `collection_tax_monthly_filings`.
    *   **Output:** Nông dân ký bằng QR code (ECDSA signature) → Trạng thái chuyển thành `ready_for_audit`.

*   **Bước 2: Kiểm toán viên ký xác nhận**
    *   **Input:** Tờ khai ở trạng thái `ready_for_audit`.
    *   **Output:** Kiểm toán viên (có khóa công khai trong `collection_auditors`) ký xác nhận → Trạng thái chuyển thành `submitted` và bị khóa (`locked: true`), không thể sửa đổi.

*   **Bước 3: Ghi vào sổ cái bất biến**
    *   **Input:** Tờ khai đã được cả nông dân và kiểm toán viên ký.
    *   **Output:** Một bản ghi bất biến được tạo trong `collection_ledger`, chứa hash của tờ khai và chữ ký của cả hai bên. Đây là bằng chứng không thể chối cãi.

---

### 🟠 **7. Gửi tới Cơ quan Thuế & Kết thúc**

*   **Input:** Bản ghi trong `collection_ledger` và `collection_tax_monthly_filings` ở trạng thái `submitted`.
*   **Output:**
    *   Cơ quan thuế xác nhận → Trạng thái `accepted`.
    *   Nếu có lỗi → Trạng thái `rejected` và hệ thống tạo bản nháp điều chỉnh mới.
    *   **Kết quả cuối cùng:** Giao dịch token và nông sản đã được **kiểm toán phi tập trung** và **chính thức được cơ quan nhà nước công nhận**, đảm bảo tuân thủ pháp luật và minh bạch tuyệt đối.

---

✅ **Kết luận:** Luồng xử lý này thể hiện một hệ sinh thái **hoàn toàn khép kín, minh bạch và tự động hóa**, trong đó **kiểm toán phi tập trung** đóng vai trò then chốt để đảm bảo mọi giao dịch — từ dữ liệu IoT đến token và nông sản — đều được xác thực bởi nhiều bên (nông dân, kiểm toán viên, blockchain, cơ quan thuế) mà không cần tin tưởng vào một thực thể trung tâm nào.
