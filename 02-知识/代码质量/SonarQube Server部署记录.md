# 环境准备
## 检查服务器是否符合 Elasticsearch

### 确认并修改服务器最大打开文件数等限制
执行以下命令检查数值是否满足要求
```bash
# 要求大于等于 524288
sysctl vm.max_map_count
# 要求大于等于 131072
sysctl fs.file-max
# 要求大于等于 131072
ulimit -n
# 要求大于等于 8192
ulimit -u
```
如果不满足要求
在 /etc/sysctl.d/99-sonarqube.conf 中录入如下内容：
```bash
 vm.max_map_count=524288
 fs.file-max=131072
```
在 /etc/security/limits.d/99-sonarqube.conf 中录入如下内容：
```bash
sonarqube   -   nofile   131072

sonarqube   -   nproc    8192
```

### 启用 seccomp
执行命令 `grep SECCOMP /boot/config-$(uname -r)` ，如果输出如下，则说明启用
```bash
CONFIG_HAVE_ARCH_SECCOMP_FILTER=y
CONFIG_SECCOMP_FILTER=y
CONFIG_SECCOMP=y
```

### 安装 JDK
```bash
yum update -y
yum install java-1.8.0-openjdk-devel -y
java -version
```

# 安装
## 安装数据库
创建专用数据库，按照顺序执行以下命令：
```bash
docker exec -it postgres bash
su - postgres psql
CREATE DATABASE sonarqube OWNER postgres;
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO postgres;
\q
```
## 通过 Docker 安装SonarQube Server
在内网 harbor 上上传镜像，具体步骤参考：[gitlab ci上无法拉取docker镜像 - EFG Q&A](http://fe-qa.dc.servyou-it.com/questions/D1S5)
启动容器之前先创建 network
```bash
docker create sonarqube-network
```
docker-compose.yaml 内容如下：
``` yaml
version: '3.4'
services:
	sonarqube:
		container_name: sonarqube
		image: 10.80.0.154:8080/base/sonarqube:lts-developer
		ports:
			- "9070:9070"
		environment:
			- SONAR_JDBC_URL=jdbc:postgresql://10.80.0.154:5432/sonarqube
			- SONAR_JDBC_USERNAME=postgres
			- SONAR_JDBC_PASSWORD=123456
			- SONAR_WEB_PORT=9070
			- SONAR_SEARCH_PORT=9071
		networks:
			- sonarqube-network
		volumes:
		    # 不能映射整个sonarqube目录，否则 docker 也会被映射到这个目录，导致服务无法启动
			- /servyouapp/sonarqube/conf:/opt/sonarqube/conf
			- /servyouapp/sonarqube/data:/opt/sonarqube/data
			- /servyouapp/sonarqube/logs:/opt/sonarqube/logs
			- /servyouapp/sonarqube/extensions:/opt/sonarqube/extensions

networks:
	sonarqube-network:
		driver: bridge
		name: sonarqube-network
		external: true
```

启动后如果遇到文件写入权限问题，执行以下命令开启权限：
```bash
sudo chmod 777 /servyouapp/sonarqube/*
```

# 扩展
## 安装插件
可选择安装以下插件：
[中文语言包](https://github.com/xuhuisheng/sonar-l10n-zh)





