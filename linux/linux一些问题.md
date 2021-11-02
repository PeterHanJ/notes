# Linux一些问题

## VIM format json

1. 安装 jq
2.  Use `:%!jq .` to format it into a human readable form.

```shell
# Fedora
sudo dnf install jq

# Ubuntu / Debian
sudo apt install jq

# CentOS / RedHat 
sudo yum install jq
```

