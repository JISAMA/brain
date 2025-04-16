# 环境准备
## 检查服务器是否符合 Elasticsearch

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
在