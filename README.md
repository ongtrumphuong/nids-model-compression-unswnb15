# Đánh giá các kỹ thuật nén mô hình học sâu cho phát hiện xâm nhập mạng hướng tới triển khai tại biên

Mã nguồn và kết quả thực nghiệm kèm theo bài báo. Bài so sánh bốn cấu hình nén cho một mô hình MLP nhẹ (4.353 tham số) phát hiện xâm nhập nhị phân trên UNSW-NB15:
- lượng tử hóa float16;
- lượng tử hóa INT8;
- cắt tỉa tham số 50%;
- cắt tỉa kết hợp float16.

Ngoài hiệu năng, kích thước và thời gian suy luận, bài phân tích ngưỡng cảnh báo và ước lượng tỉ lệ tấn công tại nơi triển khai.

## Nội dung

| Tệp / thư mục | Mô tả |
|---|---|
| `NIDS_Lightweight_UNSWNB15_v2.ipynb` | Notebook. Chạy từ tải dữ liệu đến sinh toàn bộ bảng và hình. |
| `requirements.txt` | Phiên bản thư viện của lần chạy được báo cáo (`pip freeze` trên Colab). |
| `ket_qua/` | Kết quả của lần chạy được báo cáo: bảng CSV, `ket_qua_tong_hop.json` và hình. |
| `ket_qua/cpu_info.txt` | Thông tin CPU của môi trường chạy. |
| `ket_qua/mo_hinh_seed42/` | Năm mô hình TFLite của lần chạy chính (seed 42), dạng gốc và sau gzip. |

## Môi trường

| Thành phần | Giá trị |
|---|---|
| Nền tảng | Google Colaboratory, chỉ dùng CPU |
| CPU | Intel Xeon 2,20 GHz, 2 vCPU |
| Python | 3.13 |
| TensorFlow | 2.21.0 |
| scikit-learn | 1.6.1 |
| numpy | 2.1.3 |
| pandas | 2.2.3 |

Seed được cố định và chế độ tính toán tất định của TensorFlow được bật. Mười seed dùng trong bài: 42, 123, 2024, 7, 99, 256, 512, 1001, 3407, 8888.

## Dữ liệu

Notebook tự tải bản UNSW-NB15 đã xử lý trên Kaggle ([`dhoogla/unswnb15`](https://www.kaggle.com/datasets/dhoogla/unswnb15)) qua `kagglehub`. Bản này gồm hai tệp:
- `UNSW_NB15_training-set.parquet`: 175.341 mẫu;
- `UNSW_NB15_testing-set.parquet`: 82.332 mẫu.

Bản này có 36 cột. So với tệp CSV gốc 45 cột, nó đã loại cột `id` và 8 đặc trưng: `sttl`, `dttl`, `ct_srv_src`, `ct_state_ttl`, `ct_dst_ltm`, `ct_dst_src_ltm`, `ct_src_ltm`, `ct_srv_dst`.

Kho này không chứa dữ liệu. Bản quyền dữ liệu thuộc nhóm tác giả UNSW-NB15 (Moustafa & Slay, 2015).

## Cách tái tạo kết quả

1. Mở notebook trên Google Colab. Chọn *Runtime → Change runtime type → CPU*.
2. Chạy tuần tự các ô. Ở ô tải dữ liệu, đăng nhập Kaggle khi được hỏi, hoặc đặt biến môi trường `KAGGLE_USERNAME` / `KAGGLE_KEY`.
3. Vòng lặp mười seed chiếm phần lớn thời gian chạy, khoảng 25-35 phút. Ô cuối đóng gói kết quả vào `ket_qua_v2.zip`.

## Đối chiếu bảng/hình trong bài với tệp kết quả

| Trong bài | Tệp trong `ket_qua/` |
|---|---|
| Bảng 2: hiệu năng năm cấu hình | `bang1_hieu_nang.csv` |
| Bảng 3: bootstrap ghép cặp | `bang2_bootstrap.csv` |
| Bảng 4: mười seed, ΔF1 ghép cặp | `muoi_seed_tom_tat.csv`, `muoi_seed_chi_tiet.csv` |
| So sánh ở cùng mức báo động giả (mục 4.1) | `bang10_cung_bao_dong_gia.csv` |
| Bảng 5: kích thước, gzip | `bang3_kich_thuoc.csv` |
| Bảng 6: thời gian một/bốn luồng | `bang4_thoi_gian.csv` |
| Bảng 7: quét ngưỡng trên tập kiểm thử | `bang6_quet_nguong_test.csv` |
| Bảng 8: validation và kiểm thử | `bang7_val_test.csv` |
| Bảng 9: đối chứng tái trọng số | `ket_qua_tong_hop.json`, khóa `bang8` |
| Bảng 10: cách chọn ngưỡng | `bang9_cach_chon_nguong.csv` |
| Bảng 10: ước lượng tỉ lệ tấn công | `bang_uoc_luong_pi.csv`, `mau_kiem_tra_pi.csv` |
| Phân tích độ nhạy của t* | `do_nhay_t_star.csv` |
| Hình 2 | `hinh4_so_sanh_5_cau_hinh.png` |
| Hình 3 | `hinh5_danh_doi_nguong.png` |

Ghi chú:
- Tên tệp theo thứ tự các bước trong notebook, nên không trùng số bảng trong bài.
- `hinh2_duong_cong_huan_luyen.png`, `hinh3_kich_thuoc_gzip.png` và `hinh6_roc_val_test.png` là hình bổ sung, không có trong bài.
- `bang_uoc_luong_pi.csv` và `mau_kiem_tra_pi.csv` được tính từ chính mô hình seed 42 trong `mo_hinh_seed42/`. Notebook tính lại chúng ở ô 5b.
