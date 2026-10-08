# Bản tin sáng thị trường tài chính

Trang HTML tĩnh + script Python. Không cần server, không cần Claude API.

```
index.html          giao diện, chứa khối JSON dữ liệu (được script ghi đè)
fetch_data.py       lấy dữ liệu từ các nguồn, ghi vào index.html
sources.json        chọn nguồn (provider) cho từng khối
manual_macro.json   CPI, GDP, vùng hỗ trợ/kháng cự – số liệu không có API, nhập tay
sample_data.json    dữ liệu mẫu (theo ảnh tham khảo) để xem thử giao diện
```

## Chạy

```bash
pip install yfinance feedparser requests
python fetch_data.py            # lấy dữ liệu thật
python fetch_data.py --sample   # chỉ xem giao diện với dữ liệu mẫu
```

Mở `index.html` bằng trình duyệt. Muốn tự chạy mỗi sáng 7h30 (Linux/Mac):
`30 7 * * 1-5 cd /duong/dan/bangtin && python fetch_data.py`
Windows: Task Scheduler gọi `python fetch_data.py`.

Muốn chia sẻ: đẩy `index.html` lên GitHub Pages / Netlify (kéo-thả) hoặc gửi file.

## Nguồn dữ liệu miễn phí (gợi ý)

| Khối | Nguồn chính | Thay thế |
|---|---|---|
| Chỉ số Mỹ, châu Á, dầu, vàng thế giới, DXY, EUR/USD | Yahoo Finance qua `yfinance` (miễn phí, không cần key) | Stooq (`stooq.com/q/d/l/?s=^spx&i=d` trả CSV), FRED (key miễn phí, chính thống cho lãi suất Fed/lợi suất) |
| Lợi suất TPCP Mỹ | Yahoo `^TNX`, `^TYX` | FRED `DGS10`, `DGS2`; US Treasury Fiscal Data API |
| VN-Index, thanh khoản | `"provider": "auto"` thử lần lượt TCBS → Vietcap → SSI → VNDirect (API công khai, chỉ cần `requests`); đổi thứ tự trong `"order"` | `vnstock` (đã rời PyPI, cài theo hướng dẫn tại vnstock.vn rồi đổi provider), Fireant, SSI iBoard |
| Khối ngoại | SSI iBoard (tự động) → nếu lỗi, nhập tay mục `foreign` trong `manual_macro.json` (xem CafeF/Vietstock) | Fireant, FiinTrade (cần tài khoản) |
| Tỷ giá USD/VND | Vietcombank XML công khai | NHNN (tỷ giá trung tâm, trang web), `exchangerate.host` |
| Vàng SJC | sjc.com.vn | DOJI, PNJ (trang công khai, cần parse HTML) |
| CPI, GDP, tiền tệ | GSO (gso.gov.vn), NHNN – công bố định kỳ | World Bank API (chậm hơn) |
| Tin tức | RSS CafeF, VnExpress Kinh doanh, Google News RSS (Reuters, Bloomberg) | Vietstock RSS, Tinnhanhchungkhoan RSS |

Nguồn uy tín nhưng **không có API** (dùng để đối chiếu): CafeF, Vietstock, FiinTrade, SSI iBoard, Bloomberg, Reuters.

## Đổi nguồn

Mỗi khối trong `sources.json` có `"provider"`. Ví dụ muốn lấy VN-Index bằng vnstock thay vì VNDirect:

```json
"vn_index": {"provider": "vnstock", "source": "VCI", "history_days": 6}
```

Thêm nguồn mới: viết hàm `ten_nguon(cfg)` trong `fetch_data.py` trả về dict cùng cấu trúc với hàm cũ, thêm vào `PROVIDERS`, rồi đổi tên trong `sources.json`.

Nguồn nào lỗi, khối đó để trống và tên lỗi ghi ở cuối trang (phần "Nguồn"), các khối còn lại vẫn hiện.

## Về Claude API

Mặc định phần "Nhận định nhanh" sinh theo quy tắc (`"takeaways": {"provider": "rules"}`), không tốn tiền.
Nếu muốn LLM viết nhận định tự nhiên hơn: đặt `"provider": "claude"`, cài `pip install anthropic`,
đặt biến môi trường `ANTHROPIC_API_KEY`. Script gọi Haiku 1 lần/lần chạy với dữ liệu rút gọn (~1.000 token), chi phí rất nhỏ.

## Lưu ý

- Yahoo và các trang HTML (SJC, VCB) có thể đổi cấu trúc; script đã bọc try/except, khi lỗi chỉ cần sửa hàm tương ứng.
- Dữ liệu VN sau 15h00 mới là giá đóng cửa; chạy script buổi sáng hôm sau là hợp lý.
- Đây là bản tổng hợp số liệu, không phải khuyến nghị đầu tư.
