# LLMOps: Evaluation và Thiết kế Thí nghiệm

LLMOps áp dụng kỷ luật software và measurement cho hệ thống dùng language model. Mục tiêu là giải thích được thay đổi: điều gì tốt hơn, với ai, tốn bao nhiêu và có failure mode mới nào?

## Mục tiêu học tập

- Xây offline evaluation set đại diện và tránh rò rỉ từ việc tuning.
- Đo riêng retrieval, answer quality, tool behavior, safety, latency và cost.
- Thiết kế so sánh có kiểm soát và diễn giải uncertainty.
- Dùng model judge như công cụ đo có sai lệch, không phải ground truth.
- Instrument production system nhưng hạn chế thu thập dữ liệu nhạy cảm.
- Biến evaluation thành release gate và vòng cải tiến liên tục.

## Kiến thức tiên quyết

Hoàn thành [RAG evaluation và observability](../08-nlp-llms/09-rag-evaluation-and-observability.md), [RAG worked lab](../08-nlp-llms/10-rag-system-worked-lab.md) và [Agent evaluation harness lab](../10-agentic-ai/08-agent-evaluation-harness-lab.md).

## 1. Bắt đầu bằng task contract

Ghi rõ nhiệm vụ hệ thống trước khi chọn metric:

- loại input, ngôn ngữ, user và ngữ cảnh vận hành;
- tiêu chí response đúng, đầy đủ và có căn cứ;
- lúc nào phải hỏi lại, abstain hoặc chuyển người;
- action nào được phép, action nào cần human approval;
- giới hạn latency, availability, privacy và cost;
- mức độ tác hại và failure rate chấp nhận được theo từng slice.

Tách hard constraint khỏi preference. Ví dụ latency trung bình thấp hơn không thể bù cho tool call trái quyền. Weighted score chỉ hữu ích để xếp candidate sau khi qua safety và contract constraint.

## 2. Tạo evaluation dataset đại diện cách dùng

Tạo ví dụ từ pattern tác vụ thật, case do chuyên gia viết và lỗi đã biết. Bao phủ traffic thông thường lẫn trường hợp hiếm nhưng tác động cao. Lưu ID ổn định, input, thuộc tính/evidence mong đợi, source/version, slice và ghi chú phân xử.

Các slice hữu ích:

- ngôn ngữ, domain, user type và độ dài request;
- câu hỏi trả lời được, mơ hồ, không trả lời được;
- evidence mới so với cũ;
- permission boundary và mức rủi ro;
- tool thành công, output sai, timeout và retry;
- workflow một bước so với nhiều bước.

Giữ development example để lặp, validation set để chọn configuration và locked test set cho ước lượng cuối. Không lặp đi lặp lại việc tune prompt theo locked set: như vậy biến test data thành training data. Version hóa corpus, parser, index, prompt, model và test case để tái lập kết quả.

Với dataset nhỏ, báo cả số lượng bên cạnh tỷ lệ. Nếu 2/20 ca lỗi thì observed failure là 10%, nhưng uncertainty còn lớn. Xem đúng các ví dụ và tránh khẳng định chính xác hiệu năng trên toàn population.

## 3. Đo riêng từng stage

### Retrieval

Dùng relevance label để đo Recall@k, Precision@k, reciprocal rank hoặc nDCG. Đồng thời thử access filtering, freshness, hành vi no-hit và tỷ lệ câu trả lời có supporting evidence trong retrieved set.

### Answer quality

Chấm riêng factual correctness, groundedness, completeness, citation support, abstention đã calibration và format validity. Câu trả lời trôi chảy vẫn có thể sai; câu đúng vẫn có thể thiếu căn cứ hoặc vi phạm output contract.

### Agent behavior

Đo task completion, tool selection, argument validity, call không cần thiết, phục hồi sau tool failure, policy compliance, approval handling và stopping đúng. Chấm cả trajectory lẫn final message.

### System behavior

Theo dõi stage latency, end-to-end p50/p95/p99, timeout rate, token, retry, throughput, memory và cost mỗi task thành công. Định nghĩa success trước phép chia cost: response rẻ nhưng thất bại không phải đơn vị công việc thành công.

## 4. Dùng model judge cẩn thận

Model judge giúp phân loại sơ bộ evaluation set lớn, nhưng score là ước lượng có bias. Nó có thể thích câu trả lời dài, giọng tự tin, cách diễn đạt quen thuộc hoặc cùng họ model; thứ tự trình bày có thể ảnh hưởng so sánh cặp.

Cải thiện độ tin cậy của judge:

1. Dùng rubric ngắn với tiêu chí quan sát được và ví dụ.
2. Ẩn model identity, xáo thứ tự answer khi so sánh cặp.
3. Yêu cầu kiểm tra evidence theo từng claim để chấm groundedness.
4. So mẫu quyết định judge với nhãn chuyên gia.
5. Báo disagreement, version judge, prompt và model.
6. Dùng deterministic check cho schema, citation, permission và constraint chính xác.

Không dùng một điểm judge duy nhất để quyết định release hành vi safety-critical. Chuyển slice tác động cao tới qualified human review.

## 5. Thiết kế experiment có kiểm soát

So các hệ thống trên cùng ví dụ và điều kiện. Khi có thể, chỉ đổi một yếu tố lớn: chunking, retriever, reranker, prompt, model, context budget hoặc tool policy. Nếu đổi nhiều yếu tố cùng lúc, cần factorial design có chủ đích hoặc không kết luận yếu tố nào gây kết quả.

Với paired outcome, định nghĩa chênh lệch theo từng ví dụ. Với quality score:

$$
d_i = s_{i,\mathrm{candidate}} - s_{i,\mathrm{baseline}}.
$$

Báo mean/median paired difference, confidence interval hoặc bootstrap interval, số lượng ví dụ và thay đổi theo slice. Với kết quả nhị phân, nêu số lần candidate thắng, thua, hòa và lỗi. Gain nhỏ nhưng uncertainty rộng cần thêm evidence.

Ví dụ ma trận experiment:

| Variant | Retrieval | Reranker | Câu hỏi chính |
| --- | --- | --- | --- |
| A | Dense | Không | Baseline |
| B | BM25 + dense với rank fusion | Không | Hybrid search có tăng evidence recall? |
| C | BM25 + dense với rank fusion | Cross-encoder | Reranking có tăng answer quality đủ bù latency? |

Giữ corpus snapshot, generation model, prompt và test question cố định. Cùng quality, ghi index size, retrieval latency, end-to-end latency, token count và cost.

## 6. Release gate và rollout

Đặt gate theo giai đoạn:

1. **Contract check:** schema, access control, citation ID và action bị cấm.
2. **Regression set:** failure từng gặp không được âm thầm quay lại.
3. **Quality/safety slice:** slice quan trọng đạt threshold riêng.
4. **Performance/cost:** service budget nằm trong giới hạn.
5. **Human review:** kiểm tra hành vi tác động cao hoặc mới thay đổi.
6. **Canary:** mở candidate cho traffic hạn chế, có rollback criteria.

Chọn threshold theo product risk và operational requirement; không có một accuracy number phù hợp mọi nơi. Nếu regression set rất nhỏ, coi nó là cảnh báo smoke test chứ không phải bảo đảm thống kê.

## 7. Production trace và observability tôn trọng privacy

Ghi đủ để chẩn đoán: request/trace ID, prompt/template version, model version, retrieval ID/rank, tool name và argument đã validate, timing từng stage, token count, validation outcome, retry và final status.

Raw prompt và tài liệu retrieved có thể chứa dữ liệu cá nhân, mật hoặc thuộc diện quản lý. Ưu tiên metadata và sample đã redact; hạn chế access, retention và export; tránh log secret; chỉ lấy sample full content khi cần và policy data cho phép. Trace system cũng cần access control và retention plan riêng.

Theo dõi service health và task-quality indicator:

- error, timeout và fallback rate;
- retrieval no-hit và citation-validation rate;
- abstention, escalation và tool-denial rate;
- phân phối latency và cost;
- user feedback cùng mẫu human review đến trễ;
- thay đổi request mix và hiệu năng theo slice.

Click, thumbs-up hoặc task completion là proxy, không nhất thiết là nhãn correctness. Kiểm tra selection bias: user phản hồi có thể khác user không phản hồi.

## 8. Failure analysis và vòng học hỏi

Với mỗi lỗi, ghi stage lỗi sớm nhất, impact, khả năng tái hiện, slice bị ảnh hưởng và can thiệp đề xuất. Nhóm lỗi thường gặp: source data thiếu/cũ, parsing, retrieval, context packing, generation không căn cứ, tool contract failure, permission failure, timeout hoặc evaluation defect.

Sau đó:

1. giữ failing example và trace tối thiểu;
2. tái hiện trên version hệ thống đã lưu;
3. xác định lỗi do bug, data, model behavior hay metric gap;
4. thực hiện một thay đổi có mục tiêu;
5. chạy lại regression, safety, slice, latency và cost check;
6. release dần và theo dõi rollback signal định trước.

Không xóa ví dụ khó chỉ vì version mới làm kém. Nếu case gán nhãn sai/không hợp lệ, sửa bằng adjudication có ghi nhận.

## Bài tập có đáp án

1. Hybrid retrieval tăng Recall@10 nhưng thêm 180 ms. Còn phải đo gì trước khi dùng?
2. Candidate thắng baseline theo model judge nhưng thua human review citation. Nên làm gì?
3. Vì sao production thumbs-up không tương đương expert correctness label?
4. Tuning lặp trên locked test set gây ra điều gì?
5. Thay đổi tăng average quality nhưng tăng unauthorized tool attempt. Nên đóng khung quyết định release thế nào?

### Đáp án

1. Answer grounding/correctness, phân phối latency (không chỉ trung bình), cost, index/retrieval load và các query slice liên quan.
2. Điều tra disagreement, xem claim có được source hỗ trợ không, hiệu chỉnh judge; không để judge lấn át grounding failure nghiêm trọng.
3. Feedback tự chọn và có thể phản ánh tốc độ, giọng điệu hoặc hài lòng chứ không phải độ chính xác.
4. Thông tin từ test set lọt vào quyết định thiết kế, khiến ước lượng cuối lạc quan.
5. Authorization là hard constraint. Candidate chưa đáp ứng contract cho tới khi sửa hành vi không an toàn và test lại.

## Mini-project

Chọn ứng dụng RAG hoặc dùng tool quy mô nhỏ. Xây evaluation set có version với ít nhất bốn slice, gồm request không trả lời được hoặc bị từ chối quyền. So baseline với một thay đổi trên paired example. Báo chất lượng theo stage, lỗi, uncertainty, p95 latency, cost trên mỗi task thành công và khuyến nghị release. Đưa schema trace tôn trọng privacy cùng release/rollback criteria rõ.

## Lỗi thường gặp

- Chỉ báo một quality score tổng hợp.
- So variant trên câu hỏi hoặc corpus snapshot khác nhau.
- Coi model judge là ground truth.
- Báo phần trăm thay đổi mà không nêu số mẫu/uncertainty.
- Mặc định log raw prompt vô thời hạn.
- Release theo average quality dù safety slice quan trọng giảm.
