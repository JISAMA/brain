# 环境准备
## 检查服务器是否符合 Elasticsearch

### 确认并修改服务器最大打开文件数等限制
执行以下命令检查数值是否满足要求
``` shell
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
``` shell
 vm.max_map_count=524288
 fs.file-max=131072
```
在 /etc/security/limits.d/99-sonarqube.conf 中录入如下内容：
``` shell
sonarqube   -   nofile   131072

sonarqube   -   nproc    8192
```

### 启用 seccomp
执行命令 `grep SECCOMP /boot/config-$(uname -r)` ，如果输出如下，则说明启用
``` shell
CONFIG_HAVE_ARCH_SECCOMP_FILTER=y
CONFIG_SECCOMP_FILTER=y
CONFIG_SECCOMP=y
```

### 安装 JDK
``` shell
yum update -y
yum install java-1.8.0-openjdk-devel -y
java -version
```

# 安装
## 安装数据库
创建数据库，按照顺序执行以下命令
``` shell
docker exec -it postgres bash
su - postgres psql
CREATE DATABASE sonarqube OWNER postgres;
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO postgres;
\q
```
## 通过 Docker 安装SonarQube Server
先创建 network
``` shell
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









