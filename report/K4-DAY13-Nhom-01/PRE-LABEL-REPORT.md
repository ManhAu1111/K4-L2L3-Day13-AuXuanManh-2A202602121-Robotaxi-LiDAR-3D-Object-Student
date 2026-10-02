# Báo cáo thực hành PointPillars — Day 13

Giữ bản đã điền ngoài Git, trong thư mục nhóm private do LC thu. Đây là kiểm tra formative; không ghi điểm của người khác.

## Nhóm và provenance

- Mã nhóm/phòng: Nhom-01
- Thành viên: xem `TEAMMATES.md` (Âu Xuân Mạnh - 2A202602121).
- Trạng thái: `executed-by-group`
- Người thực sự chạy; ngày/giờ; hệ máy/architecture: Âu Xuân Mạnh; 2026-10-02; amd64
- Image tag và image ID; phiên bản repo: VLearn Student Bundle latest
- PCD được cấp / frame_id; nơi được phép chạy; fingerprint nếu LC cấp: frame_id demo KITTI
- Checkpoint: PointPillars KITTI có sẵn trong image; ghi checkpoint ID/hash nếu LC cấp: default pretrained
- Phạm vi: front-window; score threshold: 0.3
- Giả định kênh thứ tư/intensity và nguồn z_ground: RGB=0, adapter kênh hằng.

## Ba lượt inference thật

A/B/C là ba lượt trên cùng PCD. Runner chạy đủ ba lượt từ một lệnh. Lấy **Số hộp** từ `n_boxes`, **mean_z** từ `mean_z` trong `run-A/B/C/summary.csv`; không tự tính lại hoặc đoán. `mean_z` không phải điểm chất lượng. Mở `side-*.png`, đối chiếu `boxes-*.json` để ghi quan sát. Số hộp không phải đáp án cần khớp nhóm khác.

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | 1 | 0.330 | `run-A/summary.csv` | Hầu như không có hộp dự đoán do sai lệch độ cao z, ảnh hưởng của dịch z trước inference là rất quan trọng. |
| B | 1.73 | 0.16 | 13 | 1.034 | `run-B/summary.csv` | Hộp xuất hiện đầy đủ 13 hộp sau khi thiết lập delta = 1.73m để đưa PCD về đúng cao độ học của model. |
| C | 1.73 | 0.32 | 6 | 1.091 | `run-C/summary.csv` | Tăng kích thước lưới grid làm giảm resolution, số hộp bị tụt từ 13 xuống còn 6 hộp (bỏ sót vật thể hoặc gộp các box). |

- A/B — chỉ đổi delta: A có 1 hộp; B có 13 hộp. Ảnh/file/vùng side-*.png khác ở cao độ các tâm. Đây là chạy lại model trên input khác, không chỉ dịch hộp cũ; điều em còn chưa chắc là liệu delta có luôn bằng 1.73 cho mọi cấu hình hay không.
- B/C — chỉ đổi pillar: B có 13 hộp; C có 6 hộp. Ảnh/file/vùng voxel_size khác ở độ mịn phân giải. Số lượng/lớp/vị trí thay đổi như sau: mất đi 7 đối tượng. Có đủ bằng chứng để kết luận tốt hơn không? Voxel nhỏ (0.16) cho kết quả tốt hơn, voxel to bị mất chi tiết.
- Giới hạn ROI và góc Side ảnh hưởng cách đọc miss/yaw thế nào? Hạn chế tầm nhìn của các vật thể bên rìa.
- JSON nào còn chưa đủ cơ sở để import? Cả A và C đều gặp vấn đề (A thiếu, C thô), B là mốc đối chiếu tốt nhất.

## Ca QC có kiểm soát — không import CVAT

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Dừng batch, kiểm từng hộp hay chưa rõ? | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | 0 / 13 | 0m | Không | Kiểm từng hộp | 13/13 hộp vẫn khớp mặt đất. |
| case-batch-z | 13 / 13 | -1.805m | Không | Dừng batch, kiểm pipeline | Tất cả hộp lơ lửng / chìm xuống cùng một lượng z = -1.805m |
| case-one-box-z | 1 / 13 | -1.805m | Không | Kiểm từng hộp | Chỉ có hộp số 1 bị lỗi, các hộp khác bình thường. |

Ghi rõ helper tạo biến đổi có chủ đích từ prediction, không phải kết quả inference riêng hoặc nhãn đúng.

## Nhận xét cá nhân

- Vai trò đã làm: Phân tích các lỗi QC và phân biệt trường hợp lỗi pipeline với lỗi cuboid lẻ.
- Một quan sát A/B/C: Thấy rõ sự khác biệt to lớn khi điều chỉnh trục z trước inference.
- Diễn giải phép z thuận/ngược: Chỉnh delta trước sẽ tác động lên toàn bộ điểm point cloud đi vào mô hình.
- Quyết định lỗi batch: Nếu mọi hộp bị lệch cao độ (như case-batch-z), cần dừng pipeline và kiểm tra logic transform, không sửa thủ công.
- Điều chưa chắc: Model chưa được train cho class Obstacle.

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca:
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:
- Nhận xét từng thành viên và quyết định dừng pipeline:
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:
