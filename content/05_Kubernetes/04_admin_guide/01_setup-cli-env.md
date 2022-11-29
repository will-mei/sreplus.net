---
weight: 1
title: "01 命令行环境配置"
date: 2022-11-22T23:59:25+08:00
draft: false
---

# 配置 kubectl 命令补全

安装 bash 自动补全工具

```shell
yum -y install bash-completion

# apt install bash-completion
```

添加 kubectl 补全脚本

```shell
kubectl completion bash >> /etc/bash_completion.d/kubectl
```

如果仅配置当前用户 bash 补全

```shell
echo 'source <(kubectl completion bash)' >>~/.bashrc 
```

重新加载 shell (重新登录/启用新 bash) 之后, kubectl 自动补全即可生效.



# 配置 vim 编辑 yaml 文件缩进

编辑 yaml 文件时, 如果使用了 tab 进行缩进会很麻烦, 下面的配置会很有帮助.

linux 环境下可以配置 vim 中将缩进宽度设置为 2 ,使用两个空格缩进而不是制表符

在 vim 中执行以下命令

```
set autoindent expandtab tabstop=2 shiftwidth=2
```

将上面的内容写入 `~/.vimrc` , 可以永久生效

