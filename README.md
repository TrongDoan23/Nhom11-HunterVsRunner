<img width="939" height="928" alt="Animation14" src="https://github.com/user-attachments/assets/be2cb928-aee4-4a11-a23f-b6a7572a4e01" />
# AI Hunter vs Runner

Trò chơi mô phỏng truy bắt trên lưới (grid) giữa hai phe đối kháng:

- **Hunter** (kẻ săn mồi): mục tiêu bắt được toàn bộ Runner trước khi hết bước.
- **Runner** (kẻ bị săn): mục tiêu sống sót đến hết số bước quy định.

Mỗi agent có thể sử dụng một trong **12 thuật toán AI** khác nhau thuộc 6 nhóm. Người dùng có thể thêm nhiều Hunter/Runner, tùy chỉnh vị trí ban đầu, vẽ vật cản và quan sát trực quan quá trình ra quyết định của từng thuật toán.

---

## Mục lục

1. [Yêu cầu hệ thống](#yêu-cầu-hệ-thống)
2. [Cài đặt](#cài-đặt)
3. [Chạy chương trình](#chạy-chương-trình)
4. [Cách chơi](#cách-chơi)
5. [Luật chơi](#luật-chơi)
6. [Các thuật toán AI](#các-thuật-toán-ai)
7. [Cấu trúc dự án](#cấu-trúc-dự-án)
8. [Cấu hình nâng cao](#cấu-hình-nâng-cao)

---

## Yêu cầu hệ thống

| Thành phần | Yêu cầu |
|---|---|
| Python | 3.9 trở lên |
| Hệ điều hành | Windows / macOS / Linux |
| Thư viện | pygame (xem requirements.txt) |
| RAM | Tối thiểu 256 MB |

---

## Cài đặt

```bash
# Clone hoặc giải nén dự án, sau đó vào thư mục gốc
pip install -r requirements.txt
```

---

## Chạy chương trình

```bash
python main.py
```

---

## Cách chơi

### Màn hình Thiết lập

| Thao tác | Mô tả |
|---|---|
| Thanh trượt **Grid size** | Thay đổi kích thước bản đồ (5×5 đến 40×40) |
| Thanh trượt **Max steps** | Số bước tối đa của trận đấu (20 đến 2000) |
| Thanh trượt **Obstacle density** | Mật độ vật cản sinh ngẫu nhiên (0%–50%) |
| Nút **Randomize** | Sinh vật cản ngẫu nhiên theo mật độ đã chọn |
| Nút **Clear Obstacles** | Xóa toàn bộ vật cản |
| **Random / Manual** | Chế độ vị trí ban đầu: tự động hoặc tự đặt tay |
| Nút **Tool: Draw Obstacles** | Bật/tắt chế độ vẽ vật cản bằng chuột lên bản đồ |
| Mũi tên `<` `>` trên card agent | Chuyển thuật toán AI cho agent đó |
| Nút **Place** | Chọn agent rồi click bản đồ để đặt vị trí thủ công |
| Nút **x** | Xóa agent khỏi danh sách |
| Nút **+ Hunter / + Runner** | Thêm agent mới (tối đa 8 mỗi loại) |
| Nút **Randomize All Positions** | Xếp vị trí ngẫu nhiên không trùng nhau cho tất cả agent |
| Nút **START MATCH** | Bắt đầu trận đấu |

### Màn hình Chơi

| Thao tác | Mô tả |
|---|---|
| Nút **Pause / Resume** hoặc phím **Space** | Tạm dừng / tiếp tục trận đấu |
| Thanh trượt **Step interval** | Tốc độ chạy (ms/bước — giá trị nhỏ hơn = nhanh hơn) |
| Nút **Play Again** | Chơi lại với cùng cấu hình |
| Nút **Menu** hoặc phím **Esc** | Quay về màn hình thiết lập |

**Ký hiệu trên bản đồ:**

- ◆ Hình thoi = Hunter
- ● Hình tròn = Runner
- Overlay màu xanh nhạt = vùng belief state của Sensorless Search
- Overlay màu cam = vùng reachable zone của AND-OR Graph Search

---

## Luật chơi

- Mỗi bước, **Hunter đi trước**, sau đó **Runner đi**.
- Sau mỗi lượt di chuyển, kiểm tra va chạm: nếu Hunter và Runner cùng ô → Runner bị bắt.
- **Hunter thắng**: tất cả Runner bị bắt trước khi hết bước.
- **Runner thắng**: còn ít nhất 1 Runner sống sót khi hết số bước tối đa.

---

## Các thuật toán AI

Dự án implement **12 thuật toán AI** thuộc 6 nhóm, mỗi nhóm phản ánh một cách tiếp cận khác nhau trong AI.

---

### Nhóm 1 — Tìm kiếm mù (Uninformed Search)

#### BFS — Breadth-First Search

**Nguyên lý:** Duyệt theo chiều rộng, mở rộng tất cả các ô ở cùng một khoảng cách trước khi đi sâu hơn.

**Hunter:** Tính đường đi ngắn nhất đến Runner gần nhất, thực hiện bước đầu tiên.

**Runner:** Dùng BFS từ vị trí Hunter để lập bản đồ khoảng cách thực toàn bản đồ, chọn ô lân cận có khoảng cách thực lớn nhất đến Hunter.

**Ưu điểm:** Đảm bảo tìm được đường đi ngắn nhất, không bị vật cản đánh lừa.

**Nhược điểm:** Tốn bộ nhớ do phải lưu toàn bộ frontier.

---

#### DFS — Depth-First Search

**Nguyên lý:** Duyệt theo chiều sâu, đi hết một nhánh trước khi thử nhánh khác.

**Hunter:** Tìm đường đến Runner gần nhất theo chiều sâu (có giới hạn depth).

**Runner:** Tương tự BFS-runner nhưng mang đặc trưng tham lam cục bộ của DFS — không đảm bảo tối ưu toàn cục.

**Ưu điểm:** Tốn ít bộ nhớ hơn BFS.

**Nhược điểm:** Không đảm bảo đường đi ngắn nhất.

---

### Nhóm 2 — Tìm kiếm có thông tin (Informed Search)

#### Greedy Best-First Search

**Nguyên lý:** Luôn mở rộng node có giá trị heuristic h(n) nhỏ nhất (không xét chi phí đã đi g(n)).

**Heuristic:** Khoảng cách Manhattan đến đích.

**Hunter:** Đi thẳng hướng có h(n) nhỏ nhất đến Runner gần nhất.

**Runner:** Chọn ô lân cận có khoảng cách thực (BFS-distance) lớn nhất đến Hunter — dùng BFS-distance thay vì Manhattan thuần để tránh bị vật cản đánh lừa.

**Ưu điểm:** Rất nhanh, ít tính toán.

**Nhược điểm:** Có thể đi vào ngõ cụt do không xét g(n).

---

#### A\* (A-Star)

**Nguyên lý:** Mở rộng node theo f(n) = g(n) + h(n), kết hợp chi phí đã đi và ước lượng còn lại.

**Heuristic:** Khoảng cách Manhattan (admissible — không overestimate).

**Hunter:** Tìm đường ngắn nhất đến Runner gần nhất, đảm bảo tối ưu.

**Runner:** Với mỗi ô lân cận, dùng A\* tính khoảng cách thực đến Hunter rồi chọn hướng xa nhất.

**Ưu điểm:** Đảm bảo tối ưu, xử lý tốt bản đồ nhiều vật cản.

**Nhược điểm:** Tốn tài nguyên hơn Greedy do phải duy trì cost_so_far.

---

### Nhóm 3 — Tìm kiếm cục bộ (Local Search)

#### Hill Climbing (với Random Restart)

**Nguyên lý:** Từ một kế hoạch ngẫu nhiên, liên tục thay thế bằng kế hoạch "hàng xóm" tốt hơn cho đến khi không cải thiện được nữa. Lặp lại từ nhiều điểm khởi đầu khác nhau (random restart) để tránh kẹt ở local optimum.

**Kế hoạch:** Chuỗi 5 bước di chuyển liên tiếp.

**Hàm fitness:** Khoảng cách thực (BFS-distance) từ vị trí cuối kế hoạch đến đối thủ — Hunter tối thiểu hóa, Runner tối đa hóa.

**Chỉ thực hiện bước đầu tiên** của kế hoạch tốt nhất tìm được (Model Predictive Control).

**Ưu điểm:** Nhanh, phù hợp real-time.

**Nhược điểm:** Không đảm bảo tối ưu toàn cục.

---

#### Local Beam Search

**Nguyên lý:** Duy trì k kế hoạch song song (beam). Mỗi vòng lặp, sinh các kế hoạch hàng xóm từ tất cả k kế hoạch, gộp lại và chọn k kế hoạch tốt nhất để tiếp tục.

**Khác biệt với Hill Climbing:** Các "tia" trong beam chia sẻ thông tin với nhau — nếu một tia tìm thấy vùng tốt, các tia khác cũng được kéo về đó.

**Ưu điểm:** Tốt hơn k lần Hill Climbing độc lập vì có sự hội tụ.

**Nhược điểm:** Tất cả tia vẫn có thể hội tụ về cùng một local optimum nếu beam quá hẹp.

---

### Nhóm 4 — Tìm kiếm thỏa mãn ràng buộc (CSP)

Mô hình hóa bài toán di chuyển K bước như một CSP:
- **Biến:** hướng đi tại mỗi bước (X₁, X₂, ..., Xₖ)
- **Miền giá trị:** 4 hướng {lên, xuống, trái, phải}
- **Ràng buộc:** không đi vào tường/vật cản, Runner không tự đi vào ô Hunter đang đứng

#### Backtracking Search

**Nguyên lý:** Gán lần lượt từng biến, khi vi phạm ràng buộc thì **quay lui** và thử giá trị khác.

**Sau khi tìm xong:** Chọn kế hoạch hợp lệ có điểm fitness tốt nhất, thực hiện bước đầu tiên.

**Ưu điểm:** Đảm bảo tìm được nghiệm nếu tồn tại.

**Nhược điểm:** Chậm nếu plan_length lớn (độ phức tạp O(4^K)).

---

#### Forward Checking

**Nguyên lý:** Giống Backtracking nhưng sau mỗi lần gán biến, **kiểm tra trước** miền giá trị của biến tiếp theo. Nếu miền rỗng (domain wipeout) → cắt tỉa nhánh ngay, không đi sâu thêm.

**Ưu điểm:** Loại bỏ sớm các nhánh thất bại — nhanh hơn Backtracking thuần, đặc biệt khi bản đồ có nhiều ngõ cụt.

**Nhược điểm:** Overhead thêm cho bước kiểm tra trước.

---

### Nhóm 5 — Tìm kiếm trong môi trường không xác định (Belief Search)

#### Sensorless Search

**Nguyên lý:** Agent **không biết chính xác vị trí đối thủ**. Thay vào đó, duy trì một *belief state* — tập hợp các vị trí có thể của đối thủ, được mở rộng dần theo thời gian (mô phỏng việc "mất dấu" đối thủ).

**Ra quyết định:** Chọn bước đi tối ưu theo **trường hợp xấu nhất** trong belief state (worst-case optimization).

**Hiển thị:** Overlay màu xanh nhạt trên bản đồ thể hiện vùng belief state.

**Ưu điểm:** Hoạt động đúng khi thông tin không đầy đủ.

**Nhược điểm:** Khi belief state quá lớn, chất lượng quyết định giảm.

---

#### AND-OR Graph Search

**Nguyên lý:** Coi đối thủ là "môi trường không xác định" — sau mỗi bước của agent (OR-node, agent được chọn tự do), đối thủ có thể di chuyển đến **bất kỳ ô lân cận nào** (AND-node, phải an toàn với **mọi** khả năng).

**Khác với Minimax:** Minimax giả định đối thủ chủ động chọn nước tệ nhất; AND-OR giả định đối thủ di chuyển hoàn toàn không xác định.

**Hiển thị:** Overlay màu cam thể hiện vùng reachable zone của đối thủ.

**Ưu điểm:** Bảo thủ và an toàn hơn Minimax trong môi trường không xác định.

**Nhược điểm:** Độ sâu bị giới hạn nhỏ do không gian tìm kiếm lớn.

---

### Nhóm 6 — Tìm kiếm đối kháng (Adversarial Search)

#### Minimax

**Nguyên lý:** Xây dựng cây game 2 người, xen kẽ lượt MAX (agent tối đa hóa lợi ích) và MIN (đối thủ tối thiểu hóa lợi ích của agent). Giả định đối thủ **luôn chơi tối ưu**.

**Hàm đánh giá:** Khoảng cách thực (BFS-distance) giữa hai agent — Hunter muốn nhỏ, Runner muốn lớn.

**Ưu điểm:** Tính đến phản ứng của đối thủ, ra quyết định chiến lược.

**Nhược điểm:** Chậm vì duyệt toàn bộ cây tìm kiếm ở mỗi độ sâu.

---

#### Alpha-Beta Pruning

**Nguyên lý:** Mở rộng từ Minimax, thêm hai biến α (giá trị tốt nhất MAX đã tìm được) và β (giá trị tốt nhất MIN đã tìm được). Cắt tỉa các nhánh chắc chắn không ảnh hưởng đến kết quả:
- **Beta cutoff:** α ≥ β tại node MAX → bỏ qua các nhánh còn lại
- **Alpha cutoff:** α ≥ β tại node MIN → bỏ qua các nhánh còn lại

**Ưu điểm:** Cùng độ sâu với Minimax nhưng xét ít node hơn đáng kể → có thể tăng độ sâu mà không tốn thêm thời gian.

**Nhược điểm:** Hiệu quả cắt tỉa phụ thuộc thứ tự duyệt node.

---

### Bảng so sánh tóm tắt

| Thuật toán | Nhóm | Đảm bảo tối ưu | Tốc độ | Tốt nhất khi |
|---|---|:---:|:---:|---|
| BFS | Mù | ✓ | Trung bình | Bản đồ phức tạp, cần đường ngắn nhất |
| DFS | Mù | ✗ | Nhanh | Bộ nhớ hạn chế |
| Greedy | Có thông tin | ✗ | Rất nhanh | Bản đồ thông thoáng |
| A\* | Có thông tin | ✓ | Trung bình | Bản đồ nhiều vật cản |
| Hill Climbing | Cục bộ | ✗ | Rất nhanh | Real-time, không gian lớn |
| Local Beam | Cục bộ | ✗ | Nhanh | Cần khám phá nhiều hướng |
| Backtracking | CSP | ✓ | Chậm | Kế hoạch nhiều bước, ràng buộc rõ |
| Forward Checking | CSP | ✓ | Trung bình | Bản đồ nhiều ngõ cụt |
| Sensorless | Không xác định | ✗ | Trung bình | Mất dấu đối thủ |
| AND-OR | Không xác định | ✗ | Chậm | Đối thủ di chuyển ngẫu nhiên |
| Minimax | Đối kháng | ✓* | Chậm | Dự đoán đối thủ, không gian nhỏ |
| Alpha-Beta | Đối kháng | ✓* | Trung bình | Như Minimax nhưng cần nhanh hơn |

*Tối ưu trong phạm vi độ sâu giới hạn.

---

## Cấu trúc dự án

```
Hunter Vs Runner/
├── main.py                   # Entry point, game loop, quản lý màn hình
├── config.py                 # Hằng số: màu sắc, kích thước, danh sách thuật toán,
│                             # tham số độ sâu Minimax/Alpha-Beta, beam width...
├── grid.py                   # Class Grid: lưu vật cản, kiểm tra walkable/border
├── agent.py                  # Class Agent: vị trí, animation, gọi thuật toán
├── game_engine.py            # Trạng thái trận đấu, luật thắng/thua, event log
├── utils.py                  # Hàm dùng chung: manhattan, get_neighbors,
│                             # random_free_cell
├── requirements.txt          # Danh sách thư viện Python cần thiết
│
├── algorithms/
│   ├── __init__.py           # Đăng ký 12 thuật toán vào ALGORITHM_FUNCS,
│   │                         # export get_decision_function()
│   ├── search_uninformed.py  # BFS, DFS
│   ├── search_informed.py    # Greedy Best-First Search, A*
│   ├── local_search.py       # Hill Climbing, Local Beam Search
│   ├── csp_search.py         # Backtracking Search, Forward Checking
│   ├── belief_search.py      # Sensorless Search, AND-OR Graph Search
│   └── adversarial.py        # Minimax, Alpha-Beta Pruning
│
├── ui/
│   ├── widgets.py            # Button, Slider, SegmentedControl, ScrollArea
│   ├── renderer.py           # Vẽ bản đồ, vật cản, agent, overlay, hiệu ứng
│   ├── hud.py                # Panel điều khiển trong màn hình chơi
│   └── menu.py               # Màn hình thiết lập trước trận
│
└── assets/
    ├── Floors/               # Texture nền
    ├── Hunter/               # Sprite animation Hunter (run/attack theo 4 hướng)
    ├── Runner/               # Sprite animation Runner
    └── Obstacles/            # Texture vật cản
```

---

## Cấu hình nâng cao

Tất cả tham số quan trọng nằm trong `config.py`:

| Tham số | Mặc định | Ý nghĩa |
|---|:---:|---|
| `MINIMAX_DEPTH` | 4 | Độ sâu cây Minimax |
| `ALPHA_BETA_DEPTH` | 6 | Độ sâu cây Alpha-Beta |
| `HC_RESTARTS` | 4 | Số lần random restart của Hill Climbing |
| `LOCAL_BEAM_WIDTH` | 6 | Số kế hoạch song song (beam width) |
| `LOCAL_BEAM_ITERATIONS` | 10 | Số vòng lặp Local Beam Search |
| `CSP_PLAN_LENGTH` | 5 | Số bước trong kế hoạch CSP |
| `AND_OR_DEPTH` | 3 | Độ sâu cây AND-OR |
| `SENSORLESS_HORIZON` | 6 | Số bước mở rộng belief state tối đa |
| `DEFAULT_GRID_SIZE` | 20 | Kích thước bản đồ mặc định |
| `DEFAULT_MAX_STEPS` | 200 | Số bước tối đa mặc định |
| `DEFAULT_OBSTACLE_DENSITY` | 0.15 | Mật độ vật cản mặc định (15%) |

Tăng `MINIMAX_DEPTH` / `ALPHA_BETA_DEPTH` để agent đánh giá sâu hơn (thông minh hơn nhưng chậm hơn). Tăng `LOCAL_BEAM_WIDTH` / `HC_RESTARTS` để tìm kiếm cục bộ chính xác hơn.
