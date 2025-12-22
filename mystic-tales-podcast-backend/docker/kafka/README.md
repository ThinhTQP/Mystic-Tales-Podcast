# 🚀 Kafka Cluster Scripts Documentation

## 📋 Tổng quan

Thư mục này chứa các script quản lý Kafka cluster cho hệ thống microservice. Có 3 file script chính:

1. **`deploy-kafka-cluster.sh`** - Script triển khai cluster
2. **`kafka-cluster-manager.sh`** - Script quản lý cluster  
3. **`setup-cluster-dirs.sh`** - Script thiết lập cấu trúc thư mục

## 🔄 Mối quan hệ với Docker Compose

### ❌ Scripts KHÔNG chạy bên trong Docker containers

**Quan trọng**: Ba script này **KHÔNG** được chạy bên trong Docker containers. Chúng là các script **host-level** được chạy trên máy host để điều khiển Docker Compose.

### 📂 Cách thức hoạt động

#### 1. **deploy-kafka-cluster.sh**
```bash
# Vị trí: ./docker/kafka/deploy-kafka-cluster.sh
# Chạy từ: Host machine (không phải trong container)
# Mục đích: Điều khiển docker-compose từ bên ngoài
```

**Cách hoạt động:**
- Script chạy trên host machine
- Chuyển đến project root: `cd ../..` (từ docker/kafka/ về root)
- Thực hiện các lệnh docker-compose:
  ```bash
  docker-compose up -d redis consul          # Khởi động infrastructure
  docker-compose up -d kafka-1 kafka-2 kafka-3  # Khởi động 3 brokers
  docker-compose up kafka-init               # Chạy topic initialization
  docker-compose up -d kafka-monitor         # Khởi động monitoring
  docker-compose up -d kafka-ui              # Khởi động UI
  ```

#### 2. **kafka-cluster-manager.sh**
```bash
# Vị trí: ./docker/kafka/kafka-cluster-manager.sh
# Chạy từ: Host machine
# Mục đích: Quản lý clusters, kiểm tra trạng thái
```

**Các chức năng:**
- `setup` - Tạo cluster mới
- `list` - Liệt kê clusters
- `info` - Thông tin cluster
- `logs` - Xem logs
- `status` - Trạng thái cluster
- `topics` - Quản lý topics

#### 3. **setup-cluster-dirs.sh**
```bash
# Vị trí: ./docker/kafka/setup-cluster-dirs.sh  
# Chạy từ: Host machine
# Mục đích: Tạo cấu trúc thư mục cho clusters
```

**Chức năng:**
- Tạo thư mục cho cluster mới
- Copy template files
- Thiết lập init và monitor scripts

### 🔗 Scripts được sử dụng BỞI Docker containers

Các script **BÊN TRONG** containers sử dụng scripts từ thư mục host thông qua volume mount:

```yaml
# Trong docker-compose.yml
volumes:
  - ./docker/kafka:/kafka:ro  # Mount thư mục kafka vào container
```

**kafka-init container:**
```bash
# Container tìm và chạy script cluster-specific:
sh "/kafka/${CLUSTER_NAME}/init-topics-cluster.sh"
# Ví dụ: sh "/kafka/kraft-cluster-1/init-topics-cluster.sh"
```

**kafka-monitor container:**
```bash
# Container tìm và chạy script monitoring:
sh "/kafka/${CLUSTER_NAME}/monitor-cluster.sh"
# Ví dụ: sh "/kafka/kraft-cluster-1/monitor-cluster.sh"
```

## 📁 Cấu trúc thư mục

```
docker/kafka/
├── deploy-kafka-cluster.sh      # 🎯 Host script - điều khiển deployment
├── kafka-cluster-manager.sh     # 🎛️ Host script - quản lý clusters  
├── setup-cluster-dirs.sh        # 📁 Host script - setup directories
├── kraft-cluster-1/             # 📂 Cluster-specific directory
│   ├── init-topics-cluster.sh   # 🔧 Container script - khởi tạo topics
│   └── monitor-cluster.sh       # 📊 Container script - monitoring
└── README.md                    # 📖 Tài liệu này
```

## 🚀 Cách sử dụng

### 1. Triển khai cluster hoàn chỉnh
```bash
# Chạy từ project root
./docker/kafka/deploy-kafka-cluster.sh
```

### 2. Quản lý cluster
```bash
# Xem danh sách clusters
./docker/kafka/kafka-cluster-manager.sh list

# Xem thông tin cluster
./docker/kafka/kafka-cluster-manager.sh info kraft-cluster-1

# Xem trạng thái
./docker/kafka/kafka-cluster-manager.sh status

# Xem logs
./docker/kafka/kafka-cluster-manager.sh logs kafka-1
```

### 3. Tạo cluster mới
```bash
# Tạo directory structure cho cluster mới
./docker/kafka/setup-cluster-dirs.sh my-new-cluster

# Tạo cluster với manager
./docker/kafka/kafka-cluster-manager.sh setup my-new-cluster
```

### 4. Chỉ khởi động Docker Compose
```bash
# Khởi động tất cả services
docker-compose up -d

# Hoặc khởi động từng nhóm
docker-compose up -d kafka-1 kafka-2 kafka-3
docker-compose up kafka-init
docker-compose up -d kafka-monitor kafka-ui
```

## ⚡ Workflow hoạt động

1. **Host scripts** (`deploy-kafka-cluster.sh`) điều khiển Docker Compose
2. **Docker Compose** khởi động containers với volume mounts
3. **Container scripts** (trong `kraft-cluster-1/`) được containers thực thi
4. **Host scripts** (`kafka-cluster-manager.sh`) quản lý và monitor

## 🔍 Troubleshooting

### Kiểm tra scripts có executable không
```bash
chmod +x docker/kafka/*.sh
chmod +x docker/kafka/kraft-cluster-1/*.sh
```

### Kiểm tra Docker Compose logs
```bash
docker-compose logs kafka-init
docker-compose logs kafka-monitor
docker-compose logs kafka-1
```

### Kiểm tra volume mounts
```bash
docker exec kafka-1 ls -la /kafka/
docker exec kafka-1 ls -la /kafka/kraft-cluster-1/
```

## 📝 Lưu ý quan trọng

- ✅ **Host scripts** chạy trên host để điều khiển Docker
- ✅ **Container scripts** chạy bên trong containers thông qua volume mounts
- ❌ **KHÔNG** chạy host scripts bên trong containers
- ❌ **KHÔNG** chạy container scripts trực tiếp trên host (trừ khi debug)

---
*Tài liệu được tạo cho Kafka 3-Broker Cluster Setup*
