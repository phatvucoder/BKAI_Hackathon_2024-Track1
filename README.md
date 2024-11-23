# BKAI_Hackathon_2024-Track1

## File structure
```bash
BKAI_Hackathon_2024-Track1/
├── data
│   ├── raw                 # data gốc từ BTC
│   │   ├── train_20041023
│   │   ├── public test
│   └── processed
│       ├── bkgr            # background được tạo ra từ ./notebooks/1.gen-bkgr.ipynb
│       ├── moving_mask     # mask được tạo ra từ ./notebooks/1.gen-bkgr.ipynb, đánh dấu vùng chuyển động (các làn đường)
│       ├── fixed_mask      # mask được tạo ra từ ./notebooks/1.gen-bkgr.ipynb, ngược lại của moving mask
│       ├── bkgr_mask       # background sau khi apply moving_mask vào
│       ├── obj_bkgr        # dán các object đã được label lên background (dùng file ./notebooks/2.objects2bkgr.ipynb)
│       ├── obj_mbkgr       # dán các object đã được label lên background (có mask)
│       └── augmented       # augmented data được sinh ra từ ./notebooks/3.augmentation.ipynb
├── docker
│   ├── Dockerfile
│   └── requirements.txt
├── inference
│   ├── 0.yolo_prediction.ipynb # test khả năng predict của model trên ảnh, toàn bộ ảnh trong folder hoặc vẽ bbox lên ảnh
│   ├── 1.gen_submit1.ipynb  # sử dụng 1 model đơn để predict tạo ra file .zip để submit lên aihub.ml
│   └── 2.gen_submit2.ipynb # sử dụng 2 model với phân loại ngày đêm để predict tạo ra file .zip để submit lên aihub.ml
├── models
│   ├── base # các code train và các file weights để thử nghiệm
│   ├── base2 # các code train và các file weights mới hơn, tốt hơn
│   └── mAP.txt # note độ chính xác của các model
└── notebooks
    ├── 0.EDA.ipynb
    ├── 1.gen-bkgr.ipynb
    ├── 2.objects2bkgr.ipynb
    ├── 3.augmentation.ipynb
    └── 4.day_night_classifier.ipynb
```

## Flow
0. Run ```docker-compose up```
1. **EDA**
    - Chạy file ./notebooks/0.EDA.ipynb
2. **Tạo background**
    - Chạy file ./notebooks/1.gen-bkgr.ipynb
    - Khuyến khích chạy trên kaggle với TPU và input dataset sau https://www.kaggle.com/datasets/phatvucoder/bkai2024 (vì cần stack hàng chục nghìn ảnh cần vài chục đến hàng trăm gb ram)
    - Nếu chạy trên local cần sửa các path sau:
        - ```data_dir = "/kaggle/input/bkai2024/daytime"``` -> ```data_dir = "../data/raw/train_20241023/daytime"```
        - ```data_dir = "/kaggle/input/bkai2024/nighttime"``` -> ```data_dir = "../data/raw/train_20241023/nighttime"```
    - Output là một file zip bên trong có các folder: bkgr, moving_mask, fixed_mask, bkgr_mask
    - Giải nén file zip trên cho các folder này vào ./data/processed
3. **Tạo bộ ảnh đã loại bỏ các object chưa được label**
    - Chạy file ./notebooks/2.objects2bkgr.ipynb
    - Cấu hình:
        - img_folder: folder chứa các ảnh gốc (có thể là ```"../data/raw/train_20241023/daytime"``` hoặc ```"../data/raw/train_20241023/nighttime"```)
        - bkgr_folder - bkgr_prefix: folder chứa background (nếu chọn background bình thường thì prefix là ```background_``` còn background đã áp mask thì prefix là ```bkgr```)
        - output_folder: folder chứa ảnh đã loại bỏ các class chưa được label
4. **Augment data**
    - Chạy file ./notebooks/3.augmentation.ipynb
    - Cấu hình:
        - IMAGE_DIR và LABEL_DIR: folder chứa ảnh gốc (ở đây sử dụng folder output từ bước 3)
        - OUTPUT_IMAGE_DIR và OUTPUT_LABEL_DIR: nơi chứa image và label của bộ data sau khi augment
5. **Chọn ra bộ input cho mô hình phân loại ngày đêm phù hợp**
    - Chạy file ./notebooks/4.day_night_classifier.ipynb
    - Class phân loại ngày đêm nhận vào 2 tham số night_intensity_ratio_threshold và brightness_threshold, nhóm cho night_intensity_ratio_threshold chạy từ 0.50->0.59 và brightness_threshold chạy từ 90->100 vì khoảng này thường cho ra kết quả cân đối giữa predict ngày và predict đêm
    - Từ đó chọn được cặp [0.54, 93]
6. **Training model**
    - Sau khi đã có data thì nhóm train model
    - Chạy các file .ipynb bên trong từ folder trong ./models/base và ./models/base (lưu ý đặt device thành 0 nếu máy chỉ có 1 gpu)
    - Thông tin chi tiết trong file mAP.txt
7. **Prediction trên tập test**
    - Chạy file 1.gen_submit1.ipynb nếu sử dụng các model base hoặc 2.gen_submit2.ipynb nếu sử dụng các model day/night
    - Cấu hình:
        - Sửa path trong ```model=YOLO("{path}") ```
        - Với {path} có dạng ```../models/{base/base2}/{type_model}/{model}.pt```

## Note
- Code có sử dụng thư viện pdatakit của em để split thông minh (phân bố đúng tỉ lệ các class vào train, val, test), đồng thời tạo các format phổ biến như yolo, coco, pascal-voc