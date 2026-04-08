# VALORANT ESPORTS SIMULATION ENGINE - SYSTEM INSTRUCTIONS

## 1. MỤC TIÊU DỰ ÁN

Dự án này là một hệ thống mô phỏng (Simulation) các giải đấu Valorant Champions Tour (VCT) theo format chuẩn của năm 2026.
Hệ thống được viết bằng Python, sử dụng lập trình hướng đối tượng (OOP). Hệ thống sẽ xuất ra các file Log (dạng JSON), và AI (Gemini) sẽ có nhiệm vụ đọc các file Log này để tạo ra bảng xếp hạng, viết bài báo phản ứng, và xây dựng cốt truyện (Storyline) cho các đội và tuyển thủ.

---

## 2. KIẾN TRÚC HƯỚNG ĐỐI TƯỢNG (OOP)

Hệ thống sử dụng các Class chính sau:

* **`Player`**: Chứa thông tin `name`, `age`. (Dùng để theo dõi chuyển nhượng/giải nghệ sau này).
* **`Team`**: Chứa `name`, `region` (Americas, EMEA, Pacific, CN), `seed` (1, 2, 3, 4), `weight` (Trọng số uy tín để bốc thăm đi giải quốc tế), và danh sách `players`.
* **`Tournament` (Class Cha)**: Quản lý định danh giải đấu (`name`, `year`, `teams_list`) và nơi xuất Log (`export_log`).
* **`Masters` (Class Con)**: Kế thừa `Tournament`. Quản lý logic giải Masters (12 đội).
* **`Champions` (Class Con)**: Kế thừa `Tournament`. Quản lý logic giải Champions (16 đội).

---

## 3. THỂ THỨC GIẢI ĐẤU & NGHIỆP VỤ (CORE LOGIC)

### 3.1. MASTERS (12 Đội)

* **Thành phần:** 4 đội Seed 1, 4 đội Seed 2, 4 đội Seed 3 (từ 4 khu vực).
* **Giai đoạn 1 - Vòng Swiss (8 đội):** Các đội Seed 2 và Seed 3 thi đấu theo hệ Thụy Sĩ. Thắng 2 trận đi tiếp, thua 2 trận bị loại. (Các đội cùng hiệu số sẽ gặp nhau: 1-0 gặp 1-0, 0-1 gặp 0-1). Chọn ra 4 đội chiến thắng.
* **Giai đoạn 2 - Playoffs (8 đội - Double Elimination):** * **NGHIỆP VỤ ĐẶC BIỆT (MANUAL PICK):** 4 đội Seed 1 sẽ được quyền **TỰ CHỌN** đối thủ từ danh sách 4 đội vượt qua vòng Swiss. (Code phải dừng lại để User nhập lựa chọn).
  * Thi đấu nhánh thắng - nhánh thua.

### 3.2. CHAMPIONS (16 Đội)

* **Thành phần:** 4 đội Seed 1, 4 đội Seed 2, 4 đội Seed 3, 4 đội Seed 4.
* **Giai đoạn 1 - GSL Groups (4 Bảng):** Mỗi bảng có 4 Seed khác nhau.
  * Trận mở màn bắt buộc: Seed 1 vs Seed 4, Seed 2 vs Seed 3.
  * Nhất và Nhì mỗi bảng đi tiếp.
* **Giai đoạn 2 - Playoffs (8 đội - Double Elimination):** Bốc thăm chéo các bảng (ví dụ: Nhất A gặp Nhì C). Không có tính năng tự chọn đối thủ.

---

## 4. LOGIC MÔ PHỎNG TRẬN ĐẤU (MATCH SIMULATION)

* **Không sử dụng Tier:** Sức mạnh của đội được xác định hoàn toàn qua `Seed` (Hạt giống).
* **Thuật toán thắng/thua cơ bản:** Dựa trên độ lệch Seed.
  * Ví dụ: Base là 50%. Mỗi bậc chênh lệch Seed cộng thêm 10% cơ hội thắng cho đội Seed cao hơn. (Seed 1 vs Seed 4 -> Chênh lệch 3 -> Seed 1 có 50 + 30 = 80% cơ hội thắng).
* **User Match (Trận của người chơi):** Nếu `is_user_match = True`, Script phải dừng lại, hiện prompt yêu cầu người chơi tự nhập tỉ số, người thắng và MVP.

---

## 5. ĐỊNH DẠNG XUẤT LOG GIAO TIẾP (JSON FORMAT)

AI chỉ đọc kết quả thông qua cấu trúc JSON này để thực hiện nhiệm vụ:

```json
{
  "tournament_name": "Masters 1 2026",
  "stage": "Playoffs - Upper Quarterfinals",
  "matches": [
    {
      "match_id": 1,
      "team_a": "EDG",
      "seed_a": 1,
      "team_b": "PRX",
      "seed_b": 2,
      "is_user_match": true,
      "score": "2-1",
      "winner": "EDG",
      "mvp": "22-EDG",
      "notable_event": "User custom info (e.g. 13-2 map 3)"
    },
    {
      "match_id": 2,
      "team_a": "SEN",
      "seed_a": 1,
      "team_b": "FNC",
      "seed_b": 3,
      "is_user_match": false,
      "score": "2-0",
      "winner": "SEN",
      "mvp": "KangKang",
      "notable_event": "Auto-simulated based on seed difference"
    }
  ]
}
```
