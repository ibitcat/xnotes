## 执行kill命令时返回 non-zero return code 错误
假设有如下 playbook：
```yaml
- hosts: local
  gather_facts: no

  tasks:
    - name: kill
      shell: kill -9 `ps -ef |grep skynet | awk '{print $2}'`
```
执行 playbook 后会提示 `non-zero return code` 错误，原因是 grep 过滤的进程有 ansible 和 grep 相关进程，可以通过运行下面的 ansible 命令来验证：
```
ansible 127.0.0.1 -m shell -a "ps -ef |grep skynet | awk '{print $2}'"
```
会发现过滤出来的进程列表中存在 `/usr/bin/python /usr/local/bin/ansible ...` 和 `bin/sh -c ps -ef ...` 这几个进程，因此需要在 grep 过滤时排除掉它们，正确的 kill 命令如下：
```shell
kill -9 `ps -ef | grep skynet | grep -v "grep\|ansible" | awk '{print $2}'`
# 或者
ps -ef | grep skynet | grep -v "grep\|ansible" | awk '{print $2}' | xargs kill -9
```