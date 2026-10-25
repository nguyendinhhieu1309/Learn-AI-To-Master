# Ensemble và boosting

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/14-ensembles-and-boosting.md)

Ensemble kết hợp nhiều predictor. Câu hỏi hữu ích không phải “nhiều model hơn có tốt hơn không?” mà là “những loại lỗi nào đang được kết hợp và cần trả giá gì?”

## Mục tiêu học tập

- Phân biệt bagging, random forest, boosting, voting và stacking.
- Giải thích vì sao lấy trung bình các tree variance cao nhưng ít tương quan có thể hữu ích.
- Suy ra một bước gradient boosting cho regression squared-error.
- So sánh model trên cùng split, metric và ngân sách tuning.
- Diễn giải feature importance mà không coi đó là bằng chứng nhân quả.

## Điều kiện tiên quyết

Trước tiên học [decision tree](12-decision-trees-and-naive-bayes.md), [validation](04-generalization-validation-and-model-selection.md) và [evaluation](02-evaluation-and-error-analysis.md).

## 1. Ý tưởng ensemble cơ bản

Với predictor $f_1,\ldots,f_M$, trung bình regression là:

$$
\hat f(x)=\frac1M\sum_{m=1}^{M}f_m(x).
$$

Nếu lỗi không tương quan hoàn toàn, lấy trung bình có thể giảm variance. Nếu mọi model mắc cùng lỗi, trung bình không đổi gì. Ensemble phức tạp có thể giảm bias hoặc cải thiện ranking, nhưng lợi ích phải đo trên dữ liệu không được dùng để tạo ensemble.

Với binary classification, soft voting lấy trung bình class probability:

$$
\hat p(y=1\mid x)=\sum_{m=1}^{M}\alpha_m \hat p_m(y=1\mid x),
\qquad \alpha_m\geq0,\quad\sum_m\alpha_m=1.
$$

Cách này giả định probability của từng model có ý nghĩa và so sánh được. Trung bình hard class label sẽ bỏ mất thông tin confidence.

## 2. Bagging và random forest

**Bagging** training mỗi model trên bootstrap sample: lấy n hàng train có hoàn lại từ n hàng gốc. Sau đó lấy trung bình prediction.

**Random forest** bổ sung feature subset ngẫu nhiên ở các split tiềm năng. Điều này giảm độ tương tự giữa tree. Mục tiêu là giữ tree riêng lẻ hữu ích nhưng giảm tương quan giữa lỗi của chúng.

Nếu mỗi tree có error variance $\sigma^2$ và tương quan lỗi từng cặp $\rho$, variance của trung bình M tree xấp xỉ:

$$
\operatorname{Var}(\bar f)=\sigma^2\left(\rho+\frac{1-\rho}{M}\right).
$$

Khi M tăng, phần không tương quan giảm nhưng phần tương quan còn nguyên. Điều này giải thích vì sao thêm tree dần có lợi ích giảm dần.

### Ước lượng out-of-bag

Mỗi bootstrap tree trung bình bỏ lại khoảng một phần ba training row. Các hàng đó có thể dùng ước lượng out-of-bag cho tree ấy. Cách này hữu ích để chẩn đoán, nhưng vẫn dựa trên training sample và không thay thế final test set chưa đụng đến.

## 3. Boosting xây dựng theo chuỗi

Boosting lần lượt thêm weak learner. Learner mới cải thiện vùng mà model hiện tại làm chưa tốt.

Với loss khả vi $L(y,F(x))$, gradient boosting fit learner tiếp theo vào negative gradient của loss theo prediction hiện tại:

$$
r_{im}=-\left.\frac{\partial L(y_i,F(x_i))}{\partial F(x_i)}\right|_{F=F_{m-1}},
\qquad
F_m(x)=F_{m-1}(x)+\eta h_m(x).
$$

$h_m$ là tree nhỏ, còn $\eta$ là learning rate. Với squared error $L=\frac12(y-F)^2$, negative gradient chính là residual $y-F$.

### Ví dụ một bước

Giả sử ba ví dụ có thứ tự với target $y=(1,3,5)$ và prediction ban đầu không đổi $F_0=3$. Residual là $(-2,0,2)$. Depth-one tree tách hai ví dụ đầu khỏi ví dụ thứ ba và dự đoán mean residual trong mỗi leaf:

$$
h(x)=(-1,-1,2).
$$

Với learning rate $\eta=0.5$:

$$
F_1=F_0+0.5h=(2.5,2.5,4).
$$

Tổng bình phương lỗi ban đầu là $(-2)^2+0^2+2^2=8$. Sau update là $(-1.5)^2+(0.5)^2+1^2=3.5$. Tree kế tiếp sẽ fit residual còn lại. Ví dụ nhỏ này minh họa update rule, không phải quy trình chọn model hoàn chỉnh.

## 4. Bias, variance và các nút regularization

Điều khiển thường gặp:

- **Tree depth hoặc số leaf:** giới hạn độ phức tạp interaction.
- **Learning rate:** thu nhỏ mỗi bước boosting.
- **Số boosting stage:** kiểm soát số lần hiệu chỉnh.
- **Row/feature subsampling:** thêm tính ngẫu nhiên.
- **Số mẫu tối thiểu mỗi leaf:** tránh leaf chỉ được hỗ trợ bởi quá ít dữ liệu.
- **Early stopping:** dùng quy trình validation dừng trước khi overfit.

Đây không phải các nút độc lập có thể vặn tùy ý. Learning rate thấp thường cần nhiều stage hơn. Chọn bằng validation plan, đừng liên tục nhìn test set.

## 5. Nên thử model nào?

| Tình huống | So sánh ban đầu phù hợp |
| --- | --- |
| Dữ liệu bảng nhỏ, sạch | linear/logistic model và shallow tree |
| Quan hệ phi tuyến trên dữ liệu bảng | random forest và gradient-boosted tree |
| Nhiều feature text thưa | thử linear model trước ensemble lớn |
| Dataset rất nhỏ | model đơn giản có regularization; validation variance cẩn thận |
| Ràng buộc latency/giải thích nghiêm ngặt | đưa model đơn giản vào nhóm so sánh |

Đây là điểm khởi đầu, không phải bảo đảm. Volume dữ liệu, missingness, kiểu feature, latency và split đánh giá đều có thể đổi kết quả.

## 6. Feature importance không tự nó là giải thích

Tree importance dựa trên impurity có thể ưu ái feature có nhiều điểm split tiềm năng. Feature tương quan có thể chia nhau hoặc che lấp importance. Permutation importance hỏi score thay đổi ra sao khi shuffle feature, nhưng feature tương quan có thể thay thế lẫn nhau khiến từng feature trông không quan trọng.

Luôn nêu:

- metric nào được đo;
- dùng partition dữ liệu nào;
- importance toàn cục hay cho subgroup;
- rủi ro correlation/leakage nào còn lại.

Importance mô tả mức model dựa vào feature theo một quy trình. Nó không cho thấy thay đổi feature sẽ khiến outcome thay đổi.

## Thí nghiệm

Trên một tabular split cố định, so sánh:

1. baseline mean hoặc majority;
2. linear model có regularization;
3. decision tree đơn;
4. random forest;
5. gradient-boosted tree.

Giữ preprocessing, metric, validation strategy và tuning budget tương đương. Ghi train/validation score, prediction latency, model size và error slice. Chỉ dùng final test sau khi đã chọn workflow.

## Bài luyện tập kèm đáp án

1. Vì sao lấy trung bình 100 tree gần như giống nhau có thể ít ích lợi?
2. Bagging và boosting khác nhau ở điểm nào?
3. Với squared-error boosting, learner mới fit target nào?
4. Nếu tree sâu hơn giảm training error nhưng tăng validation error, điều gì có thể đã đổi?
5. Vì sao impurity importance có thể phóng đại feature cardinality cao?
6. Vì sao ensemble mạnh vẫn cần so với baseline?

### Đáp án

1. Lỗi tương quan cao nên lấy trung bình không triệt tiêu được nhiều lỗi.
2. Bagging training learner độc lập trên sample resample rồi lấy trung bình; boosting tuần tự thêm learner để cải thiện fit hiện tại.
3. Residual hiện tại $y_i-F_{m-1}(x_i)$.
4. Complexity và variance tăng; model có thể đang fit pattern riêng của sample.
5. Feature đó có nhiều điểm split để thử, tăng cơ hội đạt impurity reduction có vẻ tốt.
6. Baseline cho biết độ phức tạp có mang lại lợi ích đáng kể cho metric/use case hay không.

## Lỗi thường gặp

- Gọi mọi tree ensemble là “random forest”.
- Tune số model đến khi test score cao nhất.
- Cho rằng training loss thấp chứng minh model hữu ích.
- Coi feature importance là bằng chứng nhân quả.
- So sánh model dùng split khác nhau hoặc tuning effort không ngang nhau.
