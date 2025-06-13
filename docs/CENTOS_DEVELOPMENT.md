# AIDA Server 部署文档

## 系统要求

•	Ubuntu 20.04/22.04 LTS（推荐20.04）

•	安装模式：最小化安装 + 构建工具

•	内存：最低2GB（推荐4GB）

•	存储：至少10GB可用空间


### 1.1 安装基础工具

sudo apt update

sudo apt install -y wget curl git vim unzip build-essential

安装支持 Java 8 的 JDK

sudo apt-get update && sudo apt-get install -y openjdk-8-jdk openjdk-8-jdk-headless

验证安装：

java -version

### 1.2 配置防火墙

sudo ufw allow 8084/tcp    # 后端服务端口

sudo ufw allow 3306/tcp    # MySQL数据库端口

sudo ufw enable            # 启用防火墙

sudo ufw status            # 验证端口开放状态


3. 获取项目代码
#可以从github拉不下来 可以用gitee

git clone https://github.com/realDvy/aida-server.git

git clone https://gitee.com/joey-zhou/xiaozhi-esp32-server-java.git

三、核心服务安装

1. MySQL 5.7安装

# 添加 MySQL 官方 APT 源

1.下载并安装 MySQL APT 配置工具

运行以下命令下载并安装 MySQL 的 APT 配置工具：

wget https://dev.mysql.com/get/mysql-apt-config_0.8.24-1_all.deb

sudo dpkg -i mysql-apt-config_0.8.24-1_all.deb

2.选择 MySQL 版本

安装过程中会弹出一个配置界面，要求选择 MySQL 版本。请按以下步骤操作：

使用方向键选择 MySQL Server & Cluster，然后按回车。

在子菜单中选择 mysql-5.7。

返回主菜单后，选择 OK 并按回车保存配置。

3. 更新 APT 缓存

完成配置后，更新 APT 缓存以加载新的 MySQL 源：

sudo apt update

4.安装 MySQL 5.7

运行以下命令安装 MySQL 5.7 的服务器和客户端：

sudo apt install -y mysql-server=5.7.* mysql-client=5.7.*

 
# 安装时选择5.7版本（在交互界面中选择）

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys B7B3B788A8D3785C

sudo apt update

sudo apt install -y mysql-server=5.7.* mysql-client=5.7.*

 
# 启动服务

sudo systemctl start mysql

sudo systemctl enable mysql

2. Maven安装
   
sudo apt install -y maven

mvn -v  # 验证版本（需显示3.6+）

4. Node.js 16安装
   
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -

sudo apt install -y nodejs

node -v && npm -v  # 验证版本（需显示v16.x）

6. FFmpeg安装
   
sudo apt install -y ffmpeg

ffmpeg -version  # 验证安装

四、数据库配置

1. 创建数据库用户
   
-- 登录MySQL

mysql -u root -p

-- 执行以下SQL命令

CREATE DATABASE xiaozhi CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER 'xiaozhi'@'localhost' IDENTIFIED BY 'Happy@2025';

GRANT ALL PRIVILEGES ON xiaozhi.* TO 'xiaozhi'@'localhost';

FLUSH PRIVILEGES;

exit;

3. 导入初始化数据
4. 
cd xiaozhi-esp32-server-java

mysql -u root -p xiaozhi < db/init.sql  # 输入root密码

五、语音模型部署

cd xiaozhi-esp32-server-java

wget https://alphacephei.com/vosk/models/vosk-model-cn-0.22.zip

unzip vosk-model-cn-0.22.zip

mkdir -p models && mv vosk-model-cn-0.22 models/vosk-model

六、项目部署

1. 后端部署
   
cd xiaozhi-esp32-server-java

mvn clean package -DskipTests

 
# 启动服务（后台运行）

nohup java -jar target/xiaozhi.server-1.0.jar > server.log 2>&1 &

tail -f server.log  # 观看日志

2. 前端部署
   
cd web

npm install --force  # 强制解决依赖冲突

npm run build

 
# 启动开发服务器

nohup npm run dev > frontend.log 2>&1 &

七、访问验证

访问地址：http://[公网IP]:8084

（示例：http://114.132.95.9:8084）

管理员账号：admin / 123456

八、ESP32设备配置

1.配置模型

获取apikey

模型名称:GLM-4-Plus

API URL：https://open.bigmodel.cn/api/paas/v4/

2.修改小智的服务器连接

WebSocket配置

1. 获取WS地址
   
tail -f server.log  # 查找类似日志：WebSocket started on ws://0.0.0.0:8091/ws/xiaozhi/v1/

3. 公网地址转换
   
将内网地址转换为公网地址：

 ws://[公网IP]:8091/ws/xiaozhi/v1/
 
（示例：ws://114.132.95.9:8091/ws/xiaozhi/v1/）

# 设置编译目标

idf.py set-target esp32s3

# 配置参数（需修改WS地址 删除OTA 换到WS协议）

idf.py menuconfig

# 编译

idf.py build 

# 烧录

idf.py flash monitor

九、维护操作

1. 日志管理
   
tail -f server.log        # 实时查看日志

grep "ERROR" server.log  # 过滤错误日志

3. 代码更新
   
git pull origin main

mvn clean package -DskipTests

sudo systemctl restart xiaozhi  # 需配置服务（可选）

5. 数据库备份
   
mysqldump -u root -p xiaozhi > xiaozhi_backup_$(date +%Y%m%d).sql

注意事项
公网访问：确保云服务器安全组开放 8084 和 8091 端口
MySQL兼容性：必须使用5.7版本，高版本可能不兼容
语音模型路径：确保 models/vosk-model 目录存在且包含模型文件
