# Phân Tích Dữ Liệu FMCG: Xóm Mart Global Retail Chain
**Bài phân tích dữ liệu bán lẻ FMCG đa quốc gia — Snapshot Q1/2018**
*Stack: Python (EDA & validation) → PostgreSQL (lưu trữ + Window Functions) → Power BI (visualization)*

---

## TL;DR

- **$4,332 tỷ** tổng doanh thu thực trong 4 tháng từ **6,758 triệu giao dịch** — nhưng tháng 5 sụt giảm mạnh so với đỉnh tháng 3, tín hiệu cần theo dõi ngay.
- **Confections chiếm 12,85% doanh thu** và dẫn đầu danh mục, nhưng **10 SKU cuối bảng có doanh thu gần bằng 0** — 452 SKU đang hoạt động không đồng đều nghiêm trọng.
- **97,5% Retention Rate trong tuần** trông rất cao, nhưng đây là chỉ số tuần — cần đọc đúng nghĩa: khách mua lại trong cùng 7 ngày, không phải loyalty dài hạn. Business thực chất phụ thuộc vào acquisition liên tục.
- **Vấn đề cốt lõi**: Toàn bộ 6,758M đơn hàng chỉ được xử lý bởi **23 nhân viên** — phân bổ không đều, top performer đang gánh hệ thống.
- **Cơ hội lớn nhất**: Discount đang làm mỏng margin đáng kể (gap Gross vs Net Revenue rõ ràng trên dashboard) trong khi chưa có bằng chứng về Promotion Lift thực sự.

---

## 1. DATASET, PIPELINE & GIỚI HẠN PHÂN TÍCH

### Nguồn dữ liệu & Quy mô

Dataset **fmcg_sales** từ Xóm Dataset là snapshot **4 tháng đầu năm 2018** (tháng 1–5) của chuỗi siêu thị Xóm Mart, bao gồm:

| Bảng | Số dòng | Vai trò |
|---|---|---|
| sales | 6,8M | Bảng giao dịch trung tâm — line-item từng hóa đơn |
| customers | 98,8K | Master data khách hàng thành viên |
| products | 452 | Catalog SKU với giá và metadata |
| employees | 23 | Nhân viên thu ngân |
| categories | 11 | 11 ngành hàng |
| cities | 96 | Mã địa lý cấp thành phố |
| countries | 206 | Mã địa lý cấp quốc gia |

Schema theo mô hình **Star Schema** chuẩn: bảng `sales` là Fact Table trung tâm, kết nối với 6 Dimension Table qua các khóa ngoại. Mỗi giao dịch ghi lại đầy đủ: nhân viên xử lý, khách hàng mua, sản phẩm, số lượng, chiết khấu, tổng tiền và timestamp.

### Data Pipeline

Dữ liệu được xử lý qua 3 bước:

**Bước 1 — Python (EDA & Validation):** Dùng `pandas` và `numpy` để kiểm tra data quality trước khi load. Các kiểm tra bao gồm: null values trong các khóa ngoại, giá trị âm trong `Quantity` và `TotalPrice`, orphan records (sales không match với customer/product), và phân phối outlier trong `Discount`. Kết quả: dataset tương đối sạch, không có null trong các khóa join quan trọng — đây là dataset học thuật nên data quality tốt hơn thực tế production.

**Bước 2 — PostgreSQL (Storage & Transformation):** Load toàn bộ 7 bảng vào PostgreSQL. Sau đó tạo các **View bằng Window Functions** để phục vụ Power BI

**Bước 3 — Power BI (Visualization):** Kết nối trực tiếp đến PostgreSQL, import các View đã tạo. Xây dựng 2 trang dashboard: trang **Overview** (tổng quan toàn period) và trang **Weekly Drill-down** (phân tích tuần có thể chọn).

### Giới Hạn Của Dataset Này

Trung thực về những điều **không thể kết luận** từ data:

- **Không có Cost of Goods Sold (COGS):** Cột `Discount` có nhưng không có giá vốn → không tính được Gross Margin thực, mọi phân tích về "biên lợi nhuận" chỉ là ước lượng.
- **Không có dữ liệu tồn kho:** Không phân biệt được doanh thu giảm do demand thật sự giảm hay do Out-of-Stock.
- **Mã địa lý nội bộ:** 210 "quốc gia" là mã nội bộ, không phải ISO chuẩn — không so sánh được với benchmark thị trường bên ngoài.
- **Snapshot 4 tháng:** Không đủ để xác định seasonality dài hạn (cần ít nhất 2 năm cho FMCG). Kết luận về "tháng 3 là đỉnh" cần kiểm chứng với dữ liệu năm trước.
- **Không có kênh bán hàng:** Tất cả giao dịch đều đi qua "thu ngân" — không phân biệt được online/offline, hay loại hình cửa hàng.

---

## 2. TỔNG QUAN KINH DOANH

**Xóm Mart tạo ra $4,332 tỷ doanh thu từ 6,758M giao dịch trong 4 tháng — Average Order Value $641,1 cho thấy đây là khách mua theo giỏ, không phải mua lẻ từng món.**

### Xu hướng Doanh Thu Theo Tháng

Biểu đồ Revenue by Month cho thấy một pattern đáng chú ý:

| Tháng | Doanh Thu Ước Tính | Nhận Xét |
|---|---|---|
| Tháng 1 | ~1,0 tỷ | Baseline đầu năm |
| Tháng 2 | ~1,1 tỷ | Tăng nhẹ, có thể ảnh hưởng lễ Valentine |
| Tháng 3 | ~1,2 tỷ | **Đỉnh của period** |
| Tháng 4 | ~0,9 tỷ | Sụt giảm rõ rệt |
| Tháng 5 | ~0,3 tỷ | Dữ liệu một phần (tháng chưa kết thúc?) |

Mức sụt từ tháng 3 sang tháng 4 (~25%) cần được điều tra ngay. Có 3 hypothesis cần kiểm chứng: (1) tháng 3 có chương trình khuyến mãi đặc biệt kéo demand từ tháng 4 về sớm (pull-forward effect), (2) một số thị trường địa lý lớn ngừng hoạt động, hoặc (3) đây là pattern mùa vụ bình thường của danh mục.

### Portfolio Health: 11 Ngành Hàng, Phân Bổ Không Đều

**Revenue by Category cho thấy Confections (12,85%) dẫn đầu, nhưng không ngành hàng nào vượt 15% — danh mục đang tương đối cân bằng, đây là điểm mạnh về rủi ro tập trung.**

| Ngành Hàng | % Doanh Thu | Đọc Nhanh |
|---|---|---|
| Confections | 12,85% | Dẫn đầu — tần suất mua cao, margin thấp |
| Meat | 11,38% | Top 2 — margin cao, vòng quay nhanh |
| Poultry | 10,13% | Top 3 — gần với Meat |
| Cereals | ~10% | Ổn định |
| Snails | ~10% | Đặc biệt — ngành hàng niche |
| Produce | ~9,86% | Nông sản — volatility cao |
| Beverages | ~8,59% | Thấp hơn kỳ vọng với FMCG |
| Dairy | ~8,5% | |
| Seafood | ~8,18% | |
| Meat/Poultry overlap | ~7,63% | |
| Grain | ~7,48% | Cuối bảng |

Beverages chỉ đóng góp ~8,59% là điểm đáng nghi ngờ — trong FMCG toàn cầu, Beverages thường chiếm 15–20% doanh thu bán lẻ. Cần kiểm tra lại số SKU và pricing của nhóm này.

---

## 3. PHÂN TÍCH SẢN PHẨM

**Với 452 SKU, Pareto 80/20 là câu hỏi đầu tiên cần trả lời — và Bottom 10 Products cho thấy có SKU với doanh thu chỉ 4–26 nghìn đồng trong 4 tháng. Đây là zombie SKU rõ ràng.**

### Pareto Analysis: SKU Nào Thực Sự Đang Bán?

Top 10 sản phẩm theo doanh thu (từ dashboard) đều nằm trong khoảng 8,32–8,87 triệu — phân bổ tương đối đồng đều ở nhóm đầu bảng, không có "siêu SKU" một mình gánh toàn danh mục. Đây là dấu hiệu tích cực cho sức khỏe danh mục.

Tuy nhiên, **Bottom 10 SKU** vẽ bức tranh ngược lại:

| Sản Phẩm | Doanh Thu 4 Tháng | Đánh Giá |
|---|---|---|
| Bread Cr... | ~0 nghìn | Zombie SKU — gần như không có giao dịch |
| Apricots | ~4 nghìn | Zombie SKU |
| Pastry R... | ~4 nghìn | Zombie SKU |
| Sole - Do... | ~7 nghìn | Near-zombie |
| Bread F... | ~12 nghìn | Near-zombie |
| Crab - D... | ~12 nghìn | Near-zombie |

**So What?** 6 SKU cuối bảng cộng lại không đạt 50 nghìn doanh thu trong 4 tháng. Nếu mỗi SKU tiêu tốn chi phí tồn kho, quản lý catalog, và diện tích trưng bày — đây là ứng viên hàng đầu cho SKU rationalization. Cần cross-check với `VitalityDays` trong bảng `products` để xác nhận.

### Discount Impact: Gross vs Net Revenue

Dashboard "Revenue Before and After Discount by Months" cho thấy gap rõ ràng giữa Gross Revenue và Net Revenue mỗi tháng. Ước tính discount rate trung bình ~15–20% trên toàn danh mục. Vấn đề cần làm rõ: discount này có tạo ra Incremental Volume hay chỉ là "gift margin cho khách đã có ý định mua"?

Để trả lời câu hỏi này cần so sánh volume của từng SKU trong tuần có discount vs tuần bình thường — một bài toán Window Function kinh điển với `LAG()` và `LEAD()`.

---

## 4. PHÂN TÍCH HIỆU SUẤT NHÂN VIÊN

**23 nhân viên xử lý 6,758M giao dịch trong 4 tháng — trung bình ~294K giao dịch/người. Nhưng khoảng cách giữa top và bottom performer đang tích lũy thành rủi ro vận hành.**

### Bảng Hiệu Suất Toàn Đội

| Nhân Viên | Số Đơn | Doanh Thu | Đơn/Ngày (Est.) |
|---|---|---|---|
| Wendi G. Buckley | 294,035 | $194,853,050 | ~2,450 |
| Warren C. Bartlett | 294,419 | $194,028,591 | ~2,454 |
| Tonia O. Mc Millan | 293,224 | $194,675,062 | ~2,444 |
| ... (17 nhân viên) | ~293–295K | ~$192–195M | ~2,440–2,460 |
| **Tổng** | **6,758,125** | **$4,466,293,196** | |

**Insight quan trọng nhất:** Phân bổ đơn hàng và doanh thu **gần như đồng đều tuyệt đối** — dao động chỉ ~1–2% giữa nhân viên cao nhất và thấp nhất. Đây không phải pattern tự nhiên của bán lẻ, nơi thường có 80/20 rõ rệt.

Có hai cách đọc kết quả này:

**Hypothesis 1 — Hệ thống phân công đơn tự động:** Giao dịch được assign cho nhân viên theo rotation hoặc thuật toán cân bằng tải, không phải do khách chọn hay nhân viên chủ động. Điều này giải thích sự đồng đều bất thường.

**Hypothesis 2 — Dataset được synthetic hóa:** Đây là dataset học thuật — sự phân bổ đồng đều có thể là artifact của quá trình tạo data, không phản ánh thực tế.

Dù hypothesis nào đúng, **scatter plot Employee Performance** trên dashboard tuần (Total Revenue vs Total Orders) đã visualize đúng cách để phát hiện outlier — nhân viên nào có doanh thu cao hơn đáng kể so với số lượng đơn (AOV cao hơn trung bình) là top performer thực sự.

---

## 5. PHÂN TÍCH KHÁCH HÀNG & RETENTION

**98,8K khách hàng đăng ký thành viên, AOV $641,5/tuần, Retention Rate 97,5% trong tuần 5 — nhưng con số 97,5% này cần được đọc đúng ngữ cảnh trước khi kết luận.**

### Đọc Đúng Retention Rate 97,5%

Dashboard Weekly cho thấy: **96,25K Total Customers = 96,25K Returning Customers**, với "--" ở New Customers (không có khách mới trong tuần đó). Đây là **Weekly Retention** — tỷ lệ khách đã mua tuần trước quay lại mua tuần này.

Trong FMCG bán lẻ thiết yếu (thực phẩm, đồ uống, gia vị), Weekly Retention 97,5% là **bình thường và kỳ vọng được** — khách hàng mua thực phẩm hàng tuần là behavior tự nhiên. Con số này **không nói lên nhiều về loyalty brand**, vì khách có thể đến vì tiện lợi địa lý, không phải vì gắn bó với Xóm Mart.

KPI có ý nghĩa hơn cần đo: **Cohort Retention sau 30/60/90 ngày** — trong 6,8M giao dịch này, bao nhiêu phần trăm khách mua tháng 1 quay lại mua tháng 3 hay tháng 4?

### Average LTV & Segmentation

Average LTV $2,416,1 và AOV $641,5 trong snapshot tuần 5. Nhân lên 4 tháng → LTV tổng period ước tính ~$10,000–12,000/khách hàng active. Đây là con số có ý nghĩa để tính ngưỡng CAC (Customer Acquisition Cost) tối đa chấp nhận được.

Phân khúc khách hàng theo chi tiêu (dùng NTILE(4) trong PostgreSQL):

| Phân Khúc | % Khách | Ước Tính % Doanh Thu | Hành Động |
|---|---|---|---|
| VIP (Q4 — top 25%) | 25% | ~60–65% | Giữ bằng loyalty program |
| High Value (Q3) | 25% | ~20–25% | Nâng lên VIP bằng targeted offer |
| Mid Value (Q2) | 25% | ~10–12% | Tăng basket size |
| Low Value (Q1) | 25% | ~3–5% | Reactivation hoặc chấp nhận churn |

### Purchase Interval Distribution

Biểu đồ Purchase Interval Distribution cho thấy phần lớn khách có interval **0–5 ngày** (đỉnh cao nhất), sau đó giảm dần và có đuôi dài đến 15–20 ngày. Pattern này nhất quán với hành vi mua thực phẩm: phần lớn khách mua **2–3 lần/tuần**, một nhóm nhỏ mua **2–3 lần/tháng**.

**So What?** Khách có interval > 10 ngày là ứng viên cho reactivation campaign — họ đã từng mua với tần suất cao hơn và có thể đang chuyển sang cửa hàng đối thủ.

---

## 6. PHÂN TÍCH ĐỊA LÝ

**Hơn 200 thành phố thuộc 210 quốc gia — nhưng bản đồ dashboard cho thấy tập trung chủ yếu ở United States, với các bubble lớn ở bờ Đông và Tây.**

Map visualization "Total Revenue theo City & Country" hiển thị rõ sự tập trung địa lý. Các thành phố trong bảng phân rã (Akron, Albuquerque, Anaheim, Anchorage, Arlington...) đều là thành phố Mỹ với doanh thu khá đồng đều, dao động $2,4–2,6 triệu/thành phố trong period.

Một điểm thú vị: **doanh thu theo category rất đồng đều giữa các thành phố** — không có thành phố nào lệch hơn 20% so với mix trung bình. Điều này có thể do chuẩn hóa catalog toàn chuỗi, hoặc là artifact của cách tạo data.

**City-level insight cho operations:** Với $2,4–2,6 triệu/thành phố trong 4 tháng, mỗi cửa hàng đang xử lý ~$600K–650K/tháng. Đây là benchmark để đánh giá thành phố nào đang underperform (dưới 80% benchmark → cần điều tra nguyên nhân).

---

## 7. PHÂN TÍCH GIỜ VẬN HÀNH

**"Total Orders by Hour of Day" là một trong những chart đơn giản nhưng có giá trị cao nhất cho vận hành.**

Biểu đồ dao động trong khoảng 278K–280K đơn/giờ (theo cộng dồn), với pattern gần bằng phẳng — **không có giờ cao điểm rõ rệt** trong ngày. Điều này khác biệt so với FMCG tiêu chuẩn (thường có đỉnh buổi sáng 8–9h và chiều 17–19h).

Một lần nữa, đây có thể là artifact của dataset synthetic, hoặc phản ánh thực tế: Xóm Mart phục vụ khách hàng phân bổ ở nhiều múi giờ khác nhau trên toàn cầu, làm flatten out giờ cao điểm khi nhìn ở level tổng hợp.

**Hàm ý thực tế:** Nếu là data thật, pattern phẳng này gợi ý hệ thống đang được load đồng đều 24/7 — điều này tốt cho capacity planning nhưng đặt câu hỏi về chiến lược staffing (không cần tăng nhân sự giờ cao điểm).

---

## 8. INSIGHTS TỔNG HỢP & ƯU TIÊN HÀNH ĐỘNG

| Hành Động | Impact | Effort | Ưu Tiên | KPI Đo Sau 30 Ngày |
|---|---|---|---|---|
| SKU Rationalization: loại/merge 6 zombie SKU cuối bảng | High | Low | **P0** | Số SKU active, inventory cost/SKU |
| Audit Discount Policy: đo Incremental Lift thực tế của từng chương trình | High | Medium | **P0** | Promotion Lift = (Actual - Baseline) / Baseline |
| Beverages Investigation: tại sao chỉ 8,59% khi benchmark ngành là 15–20%? | High | Low | **P1** | % Revenue by Category, số SKU Beverages |
| Cohort Analysis: đo Retention 30/60/90 ngày thật sự | High | Medium | **P1** | 30-day Cohort Retention Rate |
| Tháng 4 Drop: xác định nguyên nhân sụt 25% doanh thu | High | Medium | **P1** | MoM Revenue Change, Volume vs Price analysis |
| Mobile/UX Optimization (nếu có kênh online) | Medium | High | **P2** | CVR theo kênh, session-to-order rate |

### Quick Wins (< 2 tuần)

**Tạo Discount Effectiveness Report:** Từ View đã có trong PostgreSQL, thêm cột so sánh doanh thu tuần discount vs tuần baseline bằng `LAG()`. Kết quả: trong 2 tuần có số liệu rõ ràng về hiệu quả thực của policy chiết khấu.

**Dashboard Alert cho SKU Velocity:** Trong Power BI, thêm conditional formatting: SKU nào có doanh thu 4 tháng < $1,000 → highlight đỏ. Merchandising team có checklist để review ngay.

### Strategic Initiatives (> 1 quý)

**Loyalty Program Redesign:** Với 98,8K khách và LTV ước tính $10–12K/period, segment VIP (top 25%) đang tạo ra 60–65% doanh thu. Một loyalty program có personalized offer dựa trên purchase history có thể tăng order frequency 10–15% cho nhóm này.

**Category Expansion trong Beverages:** Nếu Beverages thực sự đang underperform so với benchmark ngành, đây là cơ hội thêm SKU hoặc renegotiate với supplier để cải thiện pricing competitiveness.

---

## 9. GIỚI HẠN PHÂN TÍCH & BƯỚC TIẾP THEO

### Ba Điều Không Thể Kết Luận Từ Dataset Này

**1. Hiệu quả thực sự của discount:** Có gap Gross vs Net Revenue rõ ràng, nhưng không có dữ liệu về volume baseline khi không có discount để tính Incremental Lift. Kết luận: "discount đang làm giảm margin" — đúng. Kết luận: "discount không hiệu quả" — chưa đủ cơ sở.

**2. Nguyên nhân drop tháng 4:** Có thể là pull-forward từ tháng 3, có thể là Out-of-Stock, có thể là mất thị trường. Với data hiện tại, không phân biệt được ba nguyên nhân này.

**3. Chất lượng trải nghiệm khách hàng:** Không có NPS, không có complaint data, không có return/refund rate — những tín hiệu quan trọng nhất về product quality và service quality trong FMCG.

### Data Gaps Cần Thu Thập Thêm

- **COGS theo SKU:** Để tính Gross Margin thực và xác định SKU có doanh thu cao nhưng margin thấp.
- **Inventory levels theo ngày:** Để phân biệt demand drop vs Out-of-Stock.
- **Return/Refund transactions:** Để đo product quality signals.
- **Dữ liệu ít nhất 2 năm:** Để xác nhận seasonality thực sự.

### Hypothesis Chưa Được Kiểm Chứng

- Tháng 3 cao hơn bình thường có phải do promotion đặc biệt không? → Cần compare volume với discount rate tháng 3 vs các tháng khác.
- Beverages underperform do thiếu SKU hay do pricing? → Cần cross-tab số SKU × average price × volume/SKU.
- Pattern phân bổ nhân viên đồng đều có phải do hệ thống round-robin không? → Cần check timestamp của giao dịch, xem có pattern shift không.

---

## Bài Học Lớn Nhất Từ Project Này

Làm xong pipeline từ Python → PostgreSQL → Power BI trên một dataset 6,8M dòng, điều tôi học được không phải là kỹ thuật Window Function hay cách config PostgreSQL connection trong Power BI — dù cả hai đều quan trọng.

Điều quan trọng hơn: **con số đúng và con số có ý nghĩa là hai thứ khác nhau hoàn toàn.**

Retention Rate 97,5% là con số đúng — query chạy ra kết quả đó, không có lỗi kỹ thuật. Nhưng nếu không hiểu nó là Weekly Retention trong một FMCG thiết yếu, bạn sẽ present lên slide cho C-level với kết luận sai. Khách hàng không trung thành — họ chỉ đang mua thực phẩm hàng tuần vì cần ăn.

Tương tự, discount gap trong dashboard trông như "vấn đề cần giải quyết ngay" — nhưng nếu discount đang tạo ra Incremental Volume đủ để bù margin mất đi, thì đó lại là chiến lược đúng. Chưa có dữ liệu baseline → chưa có kết luận.

Kỹ năng kỹ thuật giúp bạn lấy ra con số. Hiểu business model giúp bạn biết con số đó đang nói gì. Và trung thực về giới hạn của data giúp bạn không bao giờ tạo ra quyết định sai từ con số đúng.

---

*Dataset: Xóm Mart FMCG Sales — fmcg_sales (Xóm Dataset)*
*Stack: Python (pandas, numpy) → PostgreSQL (Window Functions, Views) → Power BI Desktop*
*Tác giả: El'Naut | Portfolio: https://elnautc.framer.website/*
