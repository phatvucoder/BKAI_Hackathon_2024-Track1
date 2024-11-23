# Sử dụng Python 3.11 base image
FROM python:3.11-slim

# Cập nhật hệ thống và cài đặt các dependencies cần thiết
RUN apt-get update && apt-get install -y \
    build-essential \
    python3-dev \
    git \
    && apt-get clean

# Thiết lập thư mục làm việc trong container
WORKDIR /app

# Sao chép file requirements.txt vào container
COPY requirements.txt .

# Cài đặt các thư viện từ requirements.txt
RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir -r requirements.txt

# Sao chép toàn bộ mã nguồn vào container
COPY .. /app

# Expose port cho Jupyter Notebook (nếu cần)
EXPOSE 8888

# Lệnh mặc định (có thể thay đổi)
CMD ["bash"]
