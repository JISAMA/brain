# 环境准备
## 检查服务器是否符合 Elasticsearch

执行以下命令检查数值是否满足要求（要求大于等于下面一段命令的值）
``` shell
sysctl vm.max_map_count

sysctl fs.file-max

ulimit -n

ulimit -u
```
如果不满足要求，参考下面的命令修改
``` shell

```