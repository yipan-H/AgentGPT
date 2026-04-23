FROM python:3.11-slim-bookworm as prod

# 替换为阿里云源（Debian bookworm）
RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list.d/debian.sources 2>/dev/null || \
    sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list

RUN apt-get update && apt-get install -y \
  default-libmysqlclient-dev \
  gcc \
  pkg-config \
  openjdk-17-jdk \
  build-essential \
  && rm -rf /var/lib/apt/lists/*

# 复制预先生成的 requirements.txt
COPY requirements.txt /tmp/requirements.txt

# 配置 pip 使用阿里云源，并安装依赖
RUN pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/ && \
    pip install --no-cache-dir -r /tmp/requirements.txt

# 移除编译工具（节省空间）
RUN apt-get purge -y \
  g++ \
  gcc \
  pkg-config \
  && rm -rf /var/lib/apt/lists/*

# 复制项目源代码
COPY . /app/src/
WORKDIR /app/src

# 安装项目本身（生成 metadata，解决 PackageNotFoundError）
RUN pip install -e . --no-deps

# 升级 langchain（解决 ModuleNotFoundError）
RUN pip install --upgrade langchain==0.1.0 langchain-community==0.0.10

CMD ["/usr/local/bin/python", "-m", "reworkd_platform"]

FROM prod as dev
RUN pip install pytest