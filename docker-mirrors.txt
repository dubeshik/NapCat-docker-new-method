# Docker 镜像加速器配置
# 使用方法：将下方 JSON 内容写入 /etc/docker/daemon.json，然后执行 sudo systemctl restart docker

{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerhub.icu",
    "https://docker.chenby.cn",
    "https://hub.rat.dev"
  ]
}

# NapCat 官方 Docker 镜像
# docker pull mlikiowa/napcat-docker:latest
