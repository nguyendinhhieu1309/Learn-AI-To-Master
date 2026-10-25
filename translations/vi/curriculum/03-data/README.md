# Xử lý và engineering dữ liệu: từ cơ bản đến pipeline production

[Xem trang gốc bằng tiếng Anh](../../../../curriculum/03-data/README.md)

Chất lượng và cách xử lý dữ liệu ảnh hưởng đến mọi kết quả phía sau. Module này đi từ kiểm tra/làm sạch bảng đến SQL, batch/stream pipeline đáng tin cậy, tính nhất quán của feature và hoạt động bảo đảm chất lượng dữ liệu.

## Kết quả học tập

- Kiểm tra, làm sạch, biến đổi và trực quan hóa dữ liệu thô.
- Tạo dataset bằng join an toàn, cửa sổ thời gian và feature point-in-time.
- Thiết kế batch/streaming pipeline có thể lặp lại và giám sát chất lượng.

## Điều kiện tiên quyết

Khuyến nghị biết Python cơ bản. Học NumPy và pandas song song với module.

## Thứ tự học gợi ý

1. [Làm sạch và khám phá bảng dữ liệu](01-cleaning-and-exploratory-analysis.md)
2. [Split, pipeline và data validation](02-splits-pipelines-and-validation.md)
3. [Workflow dữ liệu nâng cao](03-advanced-data-workflows.md)
4. [Làm sạch, EDA và feature engineering](04-data-cleaning-eda-and-feature-engineering.md)
5. [SQL và xây dataset](05-sql-and-dataset-construction.md)
6. [SQL database và mô hình phân tích](06-sql-databases-and-analytical-models.md)
7. [ETL/ELT pipeline và orchestration](07-etl-elt-pipelines-and-orchestration.md)
8. [Batch, streaming và tính nhất quán của feature](08-batch-streaming-and-feature-consistency.md)
9. [Data quality, feature store và streaming](09-data-quality-feature-stores-and-streaming.md)
10. [Workshop data validation và leakage](10-validation-and-leakage-workshop.md)

### Phạm vi chủ đề

- **Data model và chất lượng:** schema, grain, key, đơn vị, label, missingness, duplicate, giá trị không hợp lệ và outlier.
- **Nạp và khám phá:** CSV, JSON, Parquet, ảnh, query, phân phối, mối quan hệ, mất cân bằng và sampling bias.
- **Làm sạch và feature:** join, grouping, reshape, encoding, scaling, time feature và transformation theo domain.
- **Validation không leakage:** stratified/group/time split; preprocessing chỉ fit trên train; point-in-time join và label window.
- **SQL và tạo dataset:** analytical join, window function, cardinality query, event time, label maturity và snapshot.
- **Pipeline:** ETL/ELT idempotent, retry, checkpoint, atomic write, orchestration và backfill.
- **Batch và streaming:** event time, watermark, dữ liệu trễ, deduplication, freshness và đánh đổi throughput.
- **Vận hành feature:** offline/online parity, owner feature, schema evolution, data contract, drift signal và lineage.
- **Dữ liệu chuyên biệt:** text, image/video, time series, point cloud và sensor; giữ metadata và alignment.
- **Storage và quy mô:** chọn format, xử lý chunk và hiểu khi nào nên dùng xử lý song song/phân tán.

## Thực hành

- Audit schema dataset và báo giá trị đáng nghi trước khi làm sạch.
- So sánh mean imputation, missingness indicator và xóa hàng; nêu khi nào mỗi cách có thể gây hiểu nhầm.
- Minh họa target leakage bằng feature cố ý không hợp lệ rồi loại bỏ nó.
- Tạo pipeline có thể chạy hai lần và tạo cùng output đã validation.
- Hoàn thành [workshop data và split contract](10-validation-and-leakage-workshop.md) trước khi training trên dữ liệu phụ thuộc thời gian hoặc group.
- Giải [bộ bài tập data engineering](PROBLEM_SET_WITH_SOLUTIONS.md), tập trung vào grain của join, point-in-time correctness và retry idempotent.

## Dự án checkpoint

Chọn dataset có license mở. Viết data card ngắn, khám phá và làm sạch dữ liệu, chọn split phù hợp bài toán rồi xây preprocessing pipeline có thể lặp lại. Ghi record bị loại, giả định, rủi ro leakage và giới hạn.

## Lỗi thường gặp cần để ý

- Làm sạch toàn dataset trước khi split dù transformation có học từ dữ liệu.
- Split ngẫu nhiên quan sát phụ thuộc thời gian hoặc group.
- Xem giá trị lạ là lỗi mà không có ngữ cảnh domain.
- Trình bày biểu đồ đẹp nhưng giấu missingness, class balance hoặc sampling bias.
