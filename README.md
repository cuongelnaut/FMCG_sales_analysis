# Phân Tích Dữ Liệu FMCG: Xóm Mart Global Retail Chain
**Bài phân tích dữ liệu bán lẻ FMCG đa quốc gia — Snapshot Q1/2018**
*Stack: Python → PostgreSQL (Window Functions) → Power BI*

---

## TL;DR

- **$4,332 tỷ** doanh thu, **6,758M** giao dịch trong 4 tháng — vận hành ổn định, AOV $641 cho thấy khách mua theo giỏ hàng đầy đủ.
- Danh mục **11 ngành hàng cân bằng tốt**, nhưng tồn tại nhóm zombie SKU gần như không phát sinh giao dịch trong suốt period.
- **Beverages chỉ đạt 8,46%** — thấp hơn benchmark ngành FMCG (~15–20%) và là cơ hội tăng trưởng rõ ràng nhất.
- Discount đang làm mỏng margin đều đặn mỗi tháng, nhưng chưa có bằng chứng về Promotion Lift thực sự.

---
<img width="4150" height="3525" alt="dashboard_weekly" src="https://github.com/user-attachments/assets/e77f6473-d006-4de0-a383-67481b4f4c51" />

*Dashboard 1: Tổng quan toàn period (Jan–May 2018)*

<img width="4150" height="3337" alt="dashboard_overview" src="https://github.com/user-attachments/assets/ca237c60-d111-408e-bfbd-a50d2a7ae6be" />

*Dashboard 2: Phân tích theo tuần*
## 1. Dataset & Pipeline

Dataset **fmcg_sales** (Xóm Dataset) là snapshot 4 tháng đầu 2018 với 7 bảng, ~6,8M dòng giao dịch, 452 SKU, 98,8K khách hàng và 23 nhân viên thu ngân.

Dữ liệu được xử lý qua 3 bước: **Python** kiểm tra data quality (null, orphan records, outlier trong Discount) → **PostgreSQL** lưu trữ và tạo View bằng Window Functions (running total, customer segmentation, SKU ranking) → **Power BI** kết nối trực tiếp để visualize. Lý do tạo View ở tầng database thay vì để Power BI tự tính: tránh xử lý 6,8M dòng raw mỗi lần refresh.

**Giới hạn cần lưu ý:** Không có COGS nên không tính được Gross Margin thực. Không có dữ liệu tồn kho nên không phân biệt được demand drop vs Out-of-Stock. Snapshot 4 tháng chưa đủ để xác nhận seasonality dài hạn.

---

## 2. Tổng Quan Kinh Doanh

Doanh thu 4 tháng ổn định quanh mức ~1 tỷ/tháng:

| Tháng | Doanh Thu | Ghi Chú |
|---|---|---|
| Tháng 1 | 1,03 tỷ | Baseline |
| Tháng 2 | 0,93 tỷ | Thấp hơn do ít ngày trong tháng |
| Tháng 3 | 1,03 tỷ | Phục hồi |
| Tháng 4 | 1,00 tỷ | Ổn định |
| Tháng 5 | 0,30 tỷ | Partial data — chưa kết thúc tháng |

Không có biến động lớn — đây là tín hiệu vận hành tốt. Tháng 5 cần đánh dấu rõ là "partial" trên dashboard để tránh stakeholder đọc nhầm thành xu hướng giảm.

**Revenue by Category** cho thấy spread chỉ ~6% từ đầu đến cuối bảng — danh mục cân bằng, rủi ro tập trung thấp:

| Ngành Hàng | % Doanh Thu |
|---|---|
| Confections | 12,85% |
| Meat | 11,38% |
| Poultry | 10,16% |
| Cereals | 9,86% |
| Snails | 8,56% |
| Produce | 8,50% |
| Beverages | 8,46% |
| Dairy | 8,18% |
| Seafood | 7,63% |
| Grain | 7,48% |
| Shell fish | 6,92% |

---

## 3. Sản Phẩm

Top 10 SKU có doanh thu phân bổ tương đối đồng đều — không có SKU nào chiếm ưu thế tuyệt đối, tín hiệu tốt cho sức khỏe danh mục.

Ngược lại, Bottom 10 SKU cần xem xét các SKU ít phát sinh giao dịch trong suốt 4 tháng. Những SKU này tiêu tốn chi phí quản lý catalog và diện tích trưng bày mà không tạo ra giá trị tương xứng — ứng viên ưu tiên cho SKU rationalization.

Discount gap giữa Gross và Net Revenue nhất quán qua cả 4 tháng, gợi ý đây là flat discount policy chứ không phải chiến dịch theo thời điểm. Câu hỏi cần làm rõ: discount có đang tạo ra volume tăng thêm, hay chỉ đang transfer margin sang tay khách hàng đã có sẵn ý định mua?

---

## 4. Khách Hàng

AOV $641/đơn cho thấy khách mua giỏ hàng đầy đủ, không phải mua lẻ. Purchase Interval Distribution có đỉnh ở 0–5 ngày và đuôi kéo đến 15–20 ngày — phần lớn khách mua 2–3 lần/tuần, một nhóm nhỏ mua ít thường xuyên hơn.

Nhóm có interval tăng dần theo thời gian (từ 3 ngày lên 7 ngày lên 14 ngày) là tín hiệu churn sớm — cần can thiệp trước khi họ rời hẳn.

Dashboard tuần cung cấp Weekly Retention Rate rất cao — đây là **bình thường với FMCG thiết yếu** vì khách mua thực phẩm hàng tuần do nhu cầu sinh hoạt, không phải vì loyalty brand. KPI có ý nghĩa hơn là Cohort Retention 30/60/90 ngày, nhưng cần thêm dữ liệu để tính.

---

## 5. Nhân Viên & Địa Lý

23 nhân viên xử lý ~294K giao dịch/người trong 4 tháng với phân bổ gần như đồng đều. Trong bán lẻ thực tế, pattern này bất thường — khả năng cao giao dịch được assign theo rotation tự động hoặc là đặc điểm của dataset học thuật.

Cần xem xét lại nhân viên có tổng lượt giao dịch và doanh thu thấp, xem nguyên nhân ít phát sinh giao dịch đến từ thành phố nơi nhân viên xử ít ít lượt khách đến hay do hiệu suất của nhân viên.

Về địa lý, doanh thu tập trung ở các thành phố Mỹ với mức khá đồng đều giữa các thành phố. Đây tạo ra một benchmark vận hành rõ ràng: thành phố nào thấp hơn đáng kể là tín hiệu cần điều tra.

---

## 6. Đề Xuất Hành Động

| Hành Động | Ưu Tiên | KPI Đo Lường |
|---|---|---|
| Xác định và loại zombie SKU | P0 | Revenue/SKU, số SKU active |
| Audit discount policy — đo Incremental Lift | P0 | Promotion Lift |
| Beverages deep-dive: thiếu SKU hay pricing kém? | P1 | % Revenue Beverages |
| Thêm annotation "partial data" cho tháng 5 | P1 | — |
| Cohort Retention 30/60/90 ngày | P2 | Cohort Retention Rate |

---

## Bài Học Từ Project

Con số đúng và con số có ý nghĩa là hai thứ khác nhau. Weekly Retention rất cao là con số đúng — nhưng đọc nó như loyalty brand trong FMCG thiết yếu là kết luận sai. Tháng 5 doanh thu thấp trông như cảnh báo — nhưng thực ra là partial data.

Kỹ thuật giúp lấy ra con số. Hiểu business giúp đọc đúng con số đó.

---

*Dataset: fmcg_sales — Xóm Dataset | Stack: Python → PostgreSQL → Power BI*
*Tác giả: Cường El'Naut | Portfolio: https://elnautc.framer.website/*
