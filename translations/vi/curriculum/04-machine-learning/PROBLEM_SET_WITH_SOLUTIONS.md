# Machine learning: Bộ bài tập kèm lời giải

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/04-machine-learning/PROBLEM_SET_WITH_SOLUTIONS.md)

Workbook kiểm tra bạn biết chọn và suy luận về model phổ biến hay chỉ nhớ tên. Hãy tự làm trước khi xem lời giải. Một số câu cố ý yêu cầu nêu giả định bên cạnh phép tính.

## Bài tập

### 1. Metric từ confusion matrix

Binary classifier có TP = 18, FP = 6, FN = 12, TN = 64. Tính precision, recall, F1 và accuracy. Metric nào thay đổi nếu đổi threshold nhưng giữ nguyên ranking?

### 2. Baseline gây hiểu nhầm

Chỉ 5% ví dụ là positive. Classifier dự đoán mọi ví dụ đều negative. Tính accuracy và recall của positive class. Vì sao accuracy không đủ?

### 3. MAE và RMSE

Residual là $(1,1,8)$. Tính MAE và RMSE. Metric nào chịu ảnh hưởng của residual lớn hơn và vì sao?

### 4. Một bước gradient linear regression

Dùng $J(w,b)=\frac1{2m}\sum_i(wx_i+b-y_i)^2$ với $(0,1)$ và $(1,3)$, bắt đầu $(0,0)$.

1. Tìm gradient.
2. Cập nhật với learning rate 0.1.
3. Giải thích vì sao scale feature có thể đổi learning rate hữu dụng.

### 5. Logistic probability và loss

Với logit $z=0$, tính $\sigma(z)$. Với nhãn thật $y=1$, tính binary cross-entropy. Loss ra sao nếu model dự đoán $p=0.01$?

### 6. Scaling feature và k-NN

Euclidean distance được dùng trên dataset có thu nhập hằng năm theo nghìn (giá trị quanh 80) và tuổi theo năm (giá trị quanh 40). Vì sao thu nhập có thể chi phối việc chọn láng giềng? Nêu quy trình scaling an toàn.

### 7. Một lần cập nhật k-means

Trong một cluster có điểm $(0,0)$, $(2,0)$ và $(4,3)$. Tính centroid và giải thích vì sao k-means nhạy với khởi tạo.

### 8. Entropy và information gain

Node có ba positive và một negative. Tính entropy theo bit. Một split tạo child thuần positive size 2 và child có một positive, một negative. Tính entropy có trọng số của child và information gain.

### 9. SVM margin

Với linear SVM có $w=(3,4)$, tính geometric margin của canonical separating hyperplane. Điều gì đổi nếu nhân cả $w$ lẫn intercept với cùng hằng số dương?

### 10. Diễn giải PCA

Eigenvalue covariance matrix đã center là $(8,2,0)$. PC1 giải thích bao nhiêu phần trăm variance? Vì sao giữ 80% variance không bảo đảm giữ 80% predictive information?

### 11. Chọn split

Có phép đo lặp lại từ 30 bệnh nhân, mỗi người 20 hàng; mục tiêu dự đoán outcome cho bệnh nhân mới. Vì sao random row split gây hiểu nhầm? Nên split theo đơn vị nào?

### 12. Leakage trong preprocessing

Bạn standardize mỗi feature bằng mean và standard deviation trên toàn bộ hàng trước cross-validation. Vấn đề là gì và quy trình đúng ra sao?

### 13. So sánh model

Boosted-tree model đạt AUC 0.91, linear model đạt 0.89 trên một validation split. Cần thêm bằng chứng nào trước khi kết luận tree tốt hơn?

### 14. Threshold theo chi phí

Calibrated risk model có false-positive cost bằng 3 và false-negative cost bằng 2. Theo mô hình chi phí hai hành động, suy ra threshold xác suất.

## Lời giải chi tiết

### Lời giải 1

$$
\text{precision}=\frac{18}{18+6}=0.75,\qquad
\text{recall}=\frac{18}{18+12}=0.60.
$$

$$
F_1=\frac{2PR}{P+R}=\frac{2(0.75)(0.60)}{1.35}\approx0.667,
\qquad
\text{accuracy}=\frac{18+64}{100}=0.82.
$$

Metric phụ thuộc threshold như precision, recall, F1 và accuracy có thể đổi dù ranking-based AUC giữ nguyên.

### Lời giải 2

Accuracy là 95%; recall của positive class bằng 0. Classifier bỏ qua mọi positive. Accuracy che điều đó vì negative chiếm đa số.

### Lời giải 3

$$
\operatorname{MAE}=\frac{1+1+8}{3}=\frac{10}{3}\approx3.33,
\qquad
\operatorname{RMSE}=\sqrt{\frac{1+1+64}{3}}=\sqrt{22}\approx4.69.
$$

Bình phương khiến residual lớn nhất đóng góp không tương xứng nhiều hơn vào RMSE.

### Lời giải 4

Lúc bắt đầu, error là $-1$ và $-3$. Do đó:

$$
\frac{\partial J}{\partial w}=\frac12((-1)0+(-3)1)=-1.5,\qquad
\frac{\partial J}{\partial b}=\frac12(-1-3)=-2.
$$

Tham số sau cập nhật là $w=0.15$, $b=0.2$. Feature có giá trị lớn nhân slope gradient nên scale làm thay đổi hình học và kích thước bước thực tế.

### Lời giải 5

$\sigma(0)=1/(1+e^0)=0.5$. Với $y=1$, loss là $-\log(0.5)\approx0.693$. Với $p=0.01$, loss thành $-\log(0.01)\approx4.605$: prediction sai nhưng rất tự tin bị phạt mạnh.

### Lời giải 6

Range và đơn vị số của thu nhập có thể chi phối Euclidean distance dù tuổi hữu ích tương đương. Fit standardizer chỉ trên training data rồi áp dụng cùng transform lên validation/test. Scaling cần phù hợp bài toán; không phải model nào cũng tự động cần.

### Lời giải 7

$$
\mu=\left(\frac{0+2+4}{3},\frac{0+0+3}{3}\right)=(2,1).
$$

Center khởi đầu khác có thể gán điểm vào cluster khác và đưa đến local minimum khác. Dùng nhiều lần khởi tạo và kiểm tra tính ổn định.

### Lời giải 8

Xác suất class ở parent là $(3/4,1/4)$:

$$
H_{\mathrm{parent}}=-\frac34\log_2\frac34-\frac14\log_2\frac14\approx0.811.
$$

Child thuần có entropy 0; child cân bằng có entropy 1. Mỗi child size 2 nên entropy có trọng số là $0.5(0)+0.5(1)=0.5$. Information gain xấp xỉ $0.811-0.5=0.311$ bit.

### Lời giải 9

Với canonical margin, khoảng cách từ separating hyperplane tới support hyperplane mỗi phía là $1/\|w\|_2$. Vì $\|w\|_2=5$, margin một phía 0.2 và tổng bề rộng 0.4. Nhân $w$ cùng intercept với hằng số dương giữ nguyên decision boundary nhưng đổi canonical normalization; max-margin objective cố định scale qua constraint.

### Lời giải 10

Tổng variance là $8+2+0=10$, vì vậy component đầu giải thích $8/10=80\%$. PCA giữ variance, không nhất thiết giữ tín hiệu liên quan target. Hướng variance thấp vẫn có thể tách class hoặc chứa tín hiệu hiếm nhưng quan trọng.

### Lời giải 11

Các hàng cùng bệnh nhân chia sẻ thông tin riêng. Nếu hàng của một người có mặt trong cả train lẫn test, score có thể đo khả năng nhận biết người đã biết chứ không phải khái quát sang người mới. Group theo bệnh nhân và giữ mỗi người trong một partition.

### Lời giải 12

Fold held-out đã ảnh hưởng thống kê preprocessing được fit, nên cross-validation không còn ước lượng pipeline trên dữ liệu chưa thấy. Với mỗi fold, fit scaler trên training partition của fold rồi dùng nó transform validation partition.

### Lời giải 13

Kiểm tra độ bất định của split và nhiều fold/seed, confidence interval của metric, calibration và operating point theo tác vụ, error slice, latency, memory, chi phí, nhu cầu giải thích và tính công bằng của tuning budget. Chênh 0.02 trên một split chưa đủ kết luận.

### Lời giải 14

Can thiệp khi expected false-positive cost thấp hơn expected false-negative cost:

$$
(1-p)3 < p(2)
\quad\Longleftrightarrow\quad
p>\frac{3}{3+2}=0.6.
$$

Threshold này giả định probability được calibrated và hai chi phí phản ánh đúng đánh đổi quyết định.

## Kiểm tra mức độ thành thạo

Bạn cần giải thích giả định đằng sau mỗi phép tính, chọn split phù hợp với kết luận muốn đưa ra và bảo vệ lựa chọn metric/threshold trước khi so sánh các họ model.
