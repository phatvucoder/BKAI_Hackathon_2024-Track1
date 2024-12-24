# BKAI_Hackathon_2024-Track1

[Technical Report](Technical%20Report.pdf)

## File structure
```bash
BKAI_Hackathon_2024-Track1/
├── data
│   ├── raw                 # data gốc từ BTC
│   │   ├── train_20041023
│   │   ├── public test
│   │   └── private test
│   └── processed
│       ├── bkgr # background được tạo ra từ ./notebooks/1.gen-bkgr.ipynb
│       ├── moving_mask # mask được tạo ra từ ./notebooks/1.gen-bkgr.ipynb, đánh dấu vùng chuyển động (các làn đường)
│       ├── fixed_mask # mask được tạo ra từ ./notebooks/1.gen-bkgr.ipynb, ngược lại của moving mask
│       ├── bkgr_mask # background sau khi apply moving_mask vào
│       ├── obj_bkgr # dán các object đã được label lên background (dùng file ./notebooks/2.objects2bkgr.ipynb)
│       ├── obj_mbkgr # dán các object đã được label lên background (có mask)
│       └── augmented # augmented data được sinh ra từ ./notebooks/3.augmentation.ipynb
├── docker
│   ├── Dockerfile
│   └── requirements.txt
├── inference
│   ├── predictions # Chứa các kết quả predict từ các models, phục vụ cho ensemble
│   │   ├── dfine.txt # output từ ./inference/3.gen_submit3.ipynb
│   │   ├── yolobase.txt # output từ ./inference/1.1.gen_submit11.ipynb
│   │   └── yoloconf.txt # output từ ./inference/1.2.gen_submit12.ipynb
│   ├── 0.1.yolo_prediction.ipynb # test khả năng predict của các model YOLO trên ảnh, toàn bộ ảnh trong folder hoặc vẽ bbox lên ảnh
│   ├── 0.2.dfine_prediction.ipynb # test khả năng predict của model D-FINE trên ảnh bất kì
│   ├── 1.1.gen_submit11.ipynb  # sử dụng 1 model YOLO đơn để predict trên toàn bộ ảnh trong folder input tạo ra file .zip để submit lên aihub.ml
│   ├── 1.2.gen_submit12.ipynb  # sử dụng 1 model YOLO đơn để predict trên toàn bộ ảnh trong folder input tạo ra file .zip để submit lên aihub.ml (chỉ predict khi một bbox có confident score > 0.45 và diện tích bbox chiếm không quá 30% diện tích ảnh)
│   ├── 2.gen_submit2.ipynb # sử dụng 2 model với phân loại ngày đêm để predict tạo ra file .zip để submit lên aihub.ml
│   ├── 3.gen_submit3.ipynb # sử dụng model D-FINE để predict trên toàn bộ ảnh trong folder input tạo ra file .zip để submit lên aihub.ml
│   └── 4.ensemble.ipynb # ensemble kết quả từ các file .txt predictions trong folder ./inference/predictions để đưa ra kết quả tổng thể cuối cùng
├── models
│   ├── base # các code train YOLO và các file weights để thử nghiệm
│   ├── base2 # các code train YOLO và các file weights mới hơn, tốt hơn
│   ├── dfine # code train D-FINE và file weights
│   ├── mAP.txt # note độ chính xác của các model dựa trên mAP
│   └── mAP_F1.txt # note độ chính xác của các model dựa trên 50%mAP + 50%mAP_F1
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
4. **~~Augment data (phần này xử lý chưa tối ưu nên không dùng)~~**
    - Chạy file ./notebooks/3.augmentation.ipynb
    - Cấu hình:
        - IMAGE_DIR và LABEL_DIR: folder chứa ảnh gốc (ở đây sử dụng folder output từ bước 3)
        - OUTPUT_IMAGE_DIR và OUTPUT_LABEL_DIR: nơi chứa image và label của bộ data sau khi augment
5. **~~Chọn ra bộ input cho mô hình phân loại ngày đêm phù hợp (phần này xử lý chưa tối ưu nên không dùng)~~**
    - Chạy file ./notebooks/4.day_night_classifier.ipynb
    - Class phân loại ngày đêm nhận vào 2 tham số night_intensity_ratio_threshold và brightness_threshold, nhóm cho night_intensity_ratio_threshold chạy từ 0.50->0.59 và brightness_threshold chạy từ 90->100 vì khoảng này thường cho ra kết quả cân đối giữa predict ngày và predict đêm
    - Từ đó chọn được cặp [0.54, 93]
6. **Training model**
    - Sau khi đã có data thì nhóm train model
    - Chạy các file .ipynb bên trong từ folder trong ./models/base, ./models/base2 và ./models/dfine (lưu ý đặt device thành 0 nếu máy chỉ có 1 gpu).
    - **Trong vòng private test, nhóm dùng checkpoint của ./models/base2/base/base.pt và ./models/dfine/best_stg1.pth**
    - Log độ chính xác dựa trên metrics của BTC trong file ./models/mAP.txt và ./models/mAP_F1.txt
7. **Prediction trên tập test**
    - Chạy file ./inference/1.1.gen_submit11.ipynb, ./inference/1.2.gen_submit12.ipynb nếu sử dụng các model YOLO và ./inference/3.gen_submit3.ipynb nếu sử dụng model D-FINE
    - Cấu hình model YOLO:
        - Sửa path trong ```model=YOLO("{path}") ```
        - Với {path} có dạng ```../models/{base/base2}/{type_model}/{model}.pt```
    - Cấu hình model D-FINE: sử dụng mặc định
    - Trong vòng chung khảo, nhóm thực hiện:
        1. Chạy file ./inference/1.1.gen_submit11.ipynb:
            - Chạy toàn bộ code trong file
            - Đổi tên file .txt thành yolobase.txt và cho vào ./inference/predictions
        2. Chạy file ./inference/1.2.gen_submit12.ipynb:
            - Chạy toàn bộ code trong file
            - Đổi tên file .txt thành yoloconf.txt và cho vào ./inference/predictions
        3. Chạy file ./inference/3.gen_submit3.ipynb:
            - Chạy toàn bộ code trong file
            - Đổi tên file .txt thành dfine.txt và cho vào ./inference/predictions
        4. Chạy file ./inference/4.ensemble.ipynb
        

## Note
- Code có sử dụng thư viện pdatakit của em để split thông minh (phân bố đúng tỉ lệ các class vào train, val, test), đồng thời tạo các format phổ biến như yolo, coco, pascal-voc