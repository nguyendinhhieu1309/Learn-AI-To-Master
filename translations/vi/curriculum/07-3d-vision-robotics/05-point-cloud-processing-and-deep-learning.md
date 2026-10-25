# Xử lý point cloud và deep learning

[Xem bài gốc bằng tiếng Anh](../../../../curriculum/07-3d-vision-robotics/05-point-cloud-processing-and-deep-learning.md)

Bài này giới thiệu chuyên sâu riêng về 3D point cloud: cách biểu diễn, preprocessing, phép toán hình học, architecture học từ điểm, tác vụ, metric và failure mode phổ biến.

## 1. Point cloud chứa gì?

Point cloud là tập sample 3D, thường $p_i=(x_i,y_i,z_i)$, có thể kèm màu, intensity/reflectance, timestamp, ring index, semantic label hoặc instance ID. Khác ảnh, điểm không có thứ tự và mật độ không đều. Bề mặt gần có thể dày đặc còn phản hồi LiDAR xa thì thưa.

Luôn ghi coordinate frame, quy ước trục, đơn vị, sensor origin và timestamp. LiDAR frame, camera frame, robot base frame và world frame là các hệ tọa độ khác nhau. Cloud nhìn hợp lý vẫn có thể sai handedness hay chiều transform.

## 2. Sensor và hình học lấy mẫu

- **Spinning LiDAR:** trả range theo góc phương vị/độ cao; mật độ, khoảng cách thay đổi theo range và beam pattern.
- **Depth camera:** tạo depth map có tổ chức nhưng có giá trị thiếu, lỗi multipath và giới hạn range.
- **Stereo:** suy depth từ disparity; độ chính xác giảm theo khoảng cách.
- **Photogrammetry/MVS:** dựng điểm từ ảnh chồng lấn; có thể mơ hồ về scale nếu không có mốc tham chiếu.

Sensor noise không đồng nhất. Nếu tác vụ downstream cần độ chính xác metric, mô hình hóa uncertainty theo range và góc tới. Đồng bộ sensor trước khi fusion.

## 3. Pipeline preprocessing

Các bước thường có:

1. Đổi điểm sang reference frame được tài liệu hóa.
2. Bỏ return không hợp lệ, không finite và ngoài range.
3. Crop region of interest hoặc lọc ground nếu phù hợp.
4. Downsample để giảm compute, thường bằng voxel grid.
5. Ước lượng normal hoặc local descriptor nếu phương pháp hình học cần.
6. Giữ source ID/index khi label cần map lại về raw point.

Voxel-grid downsampling thay điểm trong một cell 3D bằng điểm đại diện. Nó giảm mật độ nhưng có thể xóa vật mảnh và boundary. Statistical outlier removal dựa trên phân phối khoảng cách tới neighbor; có thể xóa return xa thưa nhưng hợp lệ. Tune bằng kiểm tra trực quan và task metric.

### Open3D mini-lab: xem và làm sạch cloud

~~~python
import open3d as o3d

cloud = o3d.io.read_point_cloud("scene.pcd")
if cloud.is_empty():
    raise ValueError("No points loaded; check the path and file format")

# These values are in the same units as the cloud; 0.05 means 5 cm only if units are meters.
down = cloud.voxel_down_sample(voxel_size=0.05)
filtered, inlier_indices = down.remove_statistical_outlier(
    nb_neighbors=20,
    std_ratio=2.0,
)
print(f"raw={len(cloud.points)}, voxel={len(down.points)}, kept={len(filtered.points)}")
o3d.visualization.draw_geometries([filtered])
~~~

Đổi voxel size và filter setting từ từ, rồi so raw với filtered cloud. Không xem outlier removal là luôn đúng: return xa thưa có thể hợp lệ. Voxel operation của Open3D lấy trung bình các điểm trong voxel được lấp đầy; statistical filter bỏ điểm theo mean neighbor distance. Giữ index nếu annotation cần tiếp tục alignment.

## 4. Neighborhood, normal và geometric descriptor

Với từng điểm, neighborhood search tìm k nearest neighbor hoặc tất cả điểm trong radius r. Radius neighborhood phản ánh scale vật lý nhưng số điểm thay đổi theo mật độ; k-NN giữ số lượng cố định nhưng physical extent thay đổi. Normal estimation fit mặt phẳng cục bộ, thường bằng PCA của neighborhood covariance. Eigenvector ứng với eigenvalue nhỏ nhất ước lượng surface normal, nhưng hướng normal có thể mơ hồ.

Descriptor cổ điển như FPFH tóm tắt hình học cục bộ và hỗ trợ registration. Chúng là baseline hữu ích, vẫn phù hợp khi data/compute hạn chế.

## 5. Registration: ICP chi tiết hơn

Với source point $p_i$ và target match $q_i$, rigid alignment ước lượng rotation R và translation t để tối thiểu hóa:

$$
\min_{R,t}\sum_i \|Rp_i+t-q_i\|_2^2,
\qquad R^\top R=I,\;\det(R)=1
$$

ICP luân phiên tìm correspondence và ước lượng transform. Point-to-plane giảm khoảng cách theo target normal. Dùng coarse-to-fine alignment, robust loss, ngưỡng loại điểm và initial pose tốt. Kiểm tra overlap và phân bố không gian: residual thấp trên một vùng phẳng nhỏ vẫn có thể không ràng buộc đủ motion.

## 6. Neural network cho point cloud

- **PointNet:** áp shared pointwise MLP và symmetric pooling (như max) để lấy global feature bất biến với hoán vị. Xử lý điểm không thứ tự trực tiếp nhưng có thể bỏ lỡ cấu trúc cục bộ tinh.
- **PointNet++:** dựng neighborhood cục bộ phân cấp và aggregate feature ở nhiều scale, cải thiện mô hình hóa geometry cục bộ.
- **Voxel network:** rasterize không gian thành grid 3D rồi áp 3D convolution. Ý tưởng đều đặn/dễ hình dung nhưng memory tăng rất nhanh theo resolution.
- **Sparse convolution network:** chỉ tính voxel hoạt động nên xử lý scene 3D độ phân giải cao khả thi hơn khi occupancy thưa.
- **Pillar/BEV network:** gộp thông tin dọc vào cột hoặc bird's-eye-view feature, giúp LiDAR detection hiệu quả nhưng làm mất một phần cấu trúc chiều cao.
- **Point transformer:** dùng attention trên neighborhood điểm cục bộ/toàn cục để mô hình hóa quan hệ ngữ cảnh, tốn compute và memory hơn.

Lựa chọn tùy tác vụ, số điểm, resolution, hardware và latency budget. So với classical hoặc voxel baseline đơn giản.

## 7. Tác vụ 3D chính và metric

- **Classification:** một label cho object/cloud; báo accuracy và metric cân bằng class.
- **Part segmentation:** gán nhãn bộ phận; báo IoU từng class và mean IoU.
- **Semantic segmentation:** gán nhãn mọi điểm; báo class IoU, mean IoU và point accuracy, chú ý mất cân bằng.
- **3D detection:** dự đoán 3D box kèm class/confidence; đánh giá 3D IoU tại threshold đã nêu, AP theo class và range.
- **Registration:** báo translation/rotation error, inlier ratio và tỷ lệ thành công theo threshold định nghĩa.

Quy ước 3D box khác nhau: center, thứ tự size, yaw axis, coordinate frame và dimension mang nghĩa length-width-height nào. Công bố convention trước khi so metric.

## 8. Data augmentation và sampling

Augmentation thường gặp gồm rotation quanh gravity axis, translation, scale, jitter, point dropout và mô phỏng beam/range. Giữ gravity nếu tác vụ phụ thuộc vật thẳng đứng; rotation 3D tùy ý có thể tạo scene bất khả thi. Sampling ảnh hưởng object hiếm: random point sampling đều có thể bỏ vật nhỏ; class-aware sampling đổi phân phối training nên cần evaluation theo deployment distribution.

## 9. Bẫy evaluation

Split theo scene, route, drive, building hay sensor session, không split ngẫu nhiên point/crop. Kiểm tra overlap train/test, class map, coordinate transform, range truncation và label projection. Đánh giá theo distance, occlusion, point count, weather, sensor và object size. Với robotics, đo end-to-end delay và hành vi dữ liệu stale cùng geometric accuracy.

## 10. Bài tập

1. Vì sao voxel downsampling có thể xóa cột mảnh dù mật độ điểm trung bình vẫn cao?
2. Khi nào radius neighborhood tốt hơn k-NN và radius search tạo vấn đề mật độ gì?
3. Point transformer đạt score tốt nhưng vượt latency budget. Kể hai phương án representation/model khác cùng đánh đổi thông tin.
4. Detector 3D IoU tốt khi gần nhưng kém khi xa. Cần điều tra ảnh hưởng sensor/data nào?
5. Vì sao phải group train/test split theo route hoặc scene?

### Gợi ý đáp án

1. Cột mảnh có thể bị gộp hoặc mất vào voxel đại diện, dù vùng khác vẫn dày điểm.
2. Radius phù hợp khi neighborhood cần cùng physical scale; ở vùng thưa nó có thể trả về quá ít điểm, vùng dày lại có thể rất nhiều.
3. BEV/pillar tiết kiệm compute nhưng giảm chi tiết chiều cao; sparse voxel tận dụng vùng chiếm chỗ nhưng giới hạn cấu trúc không gian theo voxel.
4. LiDAR return xa thưa/noisy, disparity depth kém chính xác, occlusion, point density, range split và label quality.
5. Scene/route chung có geometry và sensor artifact tương quan, nên split ngẫu nhiên gây leakage.
