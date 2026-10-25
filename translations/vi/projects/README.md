# Danh mục Dự án

Dự án biến khái niệm thành bằng chứng đã hiểu. Bắt đầu nhỏ, xác định tác vụ và metric trước training, giữ baseline và giải thích giới hạn.

## Các mức dự án gợi ý

- **Beginner:** audit dataset và data report; tiện ích làm sạch CSV; nghiên cứu regression/classification đơn giản sau interface có tài liệu.
- **Intermediate:** ETL pipeline tái lập; image classification/detection đầu-cuối; text search; model-serving API có input validation.
- **LLM engineering:** retrieval application có evaluation retrieval/answer riêng, citation nguồn, privacy control và báo cáo cost/latency.
- **Agent engineering:** workflow LangChain hoặc LangGraph có giới hạn, skill tái sử dụng, tool hẹp, state bền vững khi cần, trace và human approval cho side effect.
- **Advanced:** RGB-D/point-cloud/ROS 2 perception; RL agent trong simulation có phân tích nhiều seed; model system có monitoring và deployment.

## Checklist viết project

- Bạn đang giải quyết vấn đề gì, cho ai?
- Dùng data nào, license gì và giới hạn ra sao?
- Chọn baseline và evaluation split nào, vì sao?
- Hệ thống sai ở đâu? Nó ứng xử thế nào khi input/setting thay đổi?
- Người khác có thể tái lập kết quả ra sao?
- Trước khi dùng trong thực tế, bạn sẽ thay đổi gì?

Bắt đầu project lớn với [AI Engineer capstone template](AI_ENGINEER_CAPSTONE_TEMPLATE.md).

Với project có hướng dẫn, theo [Core AI Engineer Capstone](CORE_AI_ENGINEER_CAPSTONE.md), trong đó có exit check theo giai đoạn, kiểm soát leakage, evaluation, packaging, monitoring và rubric chấm điểm.
