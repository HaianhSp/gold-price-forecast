# Gold Price Forecast — Dự báo giá vàng SJC Quý 4/2026

Đồ án môn **Khai phá dữ liệu** — dự báo giá vàng SJC (giá mua/giá bán) cho Quý 4/2026 dựa trên chuỗi giá lịch sử 1 năm (24/09/2025 - 24/09/2026), sử dụng kỹ thuật hồi quy (Regression) trên đặc trưng chuỗi thời gian (lag features, rolling mean).

## Mục tiêu

Từ chuỗi giá vàng SJC hàng ngày, xây dựng mô hình dự báo giá **mua** và **giá bán** cho từng ngày trong Quý 4/2026 (01/10 - 31/12), sau đó tính trung bình quý.

> Đây là bài toán **Regression / Time Series Forecasting**, khác với Classification — nên cách chia train/test và đánh giá cũng khác: chia theo thời gian (không ngẫu nhiên), và đánh giá bằng **backtest đệ quy nhiều bước** (multi-step recursive) để mô phỏng đúng kịch bản dự báo thật.

## Cấu trúc thư mục

```
gold-price-forecast/
├── data/
│   └── vietdataverse_gold_preview.csv   # Giá vàng SJC hàng ngày: date, buy_prices, sell_prices
├── notebooks/
│   └── gold_price_forecast_q4_2026.ipynb
├── .gitignore
└── README.md
```

## Dữ liệu

- **Nguồn**: `vietdataverse_gold_preview.csv` — 361 dòng, 3 cột (`date`, `buy_prices`, `sell_prices`), từ 24/09/2025 đến 24/09/2026.
- **Vấn đề dữ liệu đã xử lý**:
  - Thiếu 5 ngày trong chuỗi (23-27/04/2026) → điền bằng nội suy tuyến tính (`interpolate`).
  - File có BOM ở đầu → đọc bằng `encoding='utf-8-sig'`.

## Phương pháp

| Bước | Mô tả |
|---|---|
| Feature engineering | Lag features (1, 2, 3, 7, 14, 30 ngày trước), rolling mean (7/14/30 ngày), đặc trưng lịch (thứ, ngày, tháng) |
| Chia dữ liệu | Theo thời gian: train là quá khứ, test là 90 ngày gần nhất — **không** chia ngẫu nhiên (tránh nhìn trước tương lai) |
| Dự báo đệ quy | Dự báo từng ngày, dùng chính giá trị vừa dự báo làm lag input cho ngày kế tiếp (multi-step recursive forecast) |
| Mô hình so sánh | Naive (giữ nguyên giá cuối), Linear Regression, Random Forest Regressor |
| Đánh giá | MAE, RMSE, MAPE trên backtest 90 ngày (mô phỏng đúng độ dài dự báo thật cho Quý 4) |

## Kết quả

**Backtest 90 ngày (26/06 - 24/09/2026):**

| Mô hình | MAE | RMSE | MAPE |
|---|---|---|---|
| Naive (giữ nguyên giá) | 2.45 triệu | 3.05 triệu | **1.69%** |
| Linear Regression | 11.37 triệu | 12.46 triệu | 7.84% |
| Random Forest | 3.34 triệu | 4.06 triệu | 2.31% |

**Phát hiện quan trọng**: Naive baseline thắng cả 2 mô hình ML trong giai đoạn backtest — vì 3 tháng gần đây giá vàng đi ngang khá ổn định (142-146 triệu), nên "giữ nguyên giá" khó bị đánh bại. Đây là insight thật từ dữ liệu, không phải hạn chế của việc thực hiện.

**Dự báo Quý 4/2026 (dùng Random Forest — tốt nhất trong 2 mô hình ML):**
- Giá mua trung bình dự báo: **~142.98 triệu đồng/lượng**
- Giá bán trung bình dự báo: **~145.36 triệu đồng/lượng**
- Khoảng dao động dự kiến: 141.5 - 148 triệu (giá bán)

## Giới hạn của mô hình

- Chỉ dùng chính chuỗi giá quá khứ, không có dữ liệu vĩ mô (giá vàng thế giới, tỷ giá USD/VND, lãi suất, chính sách NHNN...) — vốn ảnh hưởng lớn đến giá vàng SJC thực tế.
- Dự báo 98 ngày là khá xa nên sai số tích lũy đáng kể (đặc điểm cố hữu của multi-step recursive forecasting).
- Kết quả nên được xem là **ước tính có cơ sở dữ liệu**, không phải khuyến nghị đầu tư.

## Cách chạy

```bash
pip install pandas numpy scikit-learn matplotlib seaborn jupyter
```

Mở và chạy toàn bộ (Run All):

```bash
jupyter notebook notebooks/gold_price_forecast_q4_2026.ipynb
```

## Kỹ thuật khai phá dữ liệu áp dụng

- **Data Cleaning**: xử lý ngày thiếu, xử lý encoding.
- **Feature Engineering cho chuỗi thời gian**: lag features, rolling mean, đặc trưng lịch.
- **Regression**: Linear Regression, Random Forest Regressor.
- **Time Series Evaluation**: backtest đệ quy, so sánh với naive baseline, MAE/RMSE/MAPE.

## Hướng mở rộng (nếu có thời gian)

- Thử ARIMA/SARIMA hoặc Prophet — các mô hình chuyên cho chuỗi thời gian, bắt mùa vụ tốt hơn.
- Bổ sung dữ liệu ngoại sinh: giá vàng thế giới (XAU/USD), tỷ giá USD/VND làm feature.

## Tác giả

Lê Nguyễn Hải Anh — Phenikaa University
