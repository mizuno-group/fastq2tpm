# 1. desired environment 
## condaの入ったbase image
FROM continuumio/miniconda3

# linux setting
RUN mkdir /workspace

# Dockerの作成にキー入力を使わない。
ENV DEBIAN_FRONTEND=noninteractive

# sudoのインストール(必要ない?)
RUN apt-get update && apt-get install -y sudo

## Docker内で使用する名前。何でもよい
ARG USERNAME=vscode
ARG USER_UID=1234
ARG USER_GID=$USER_UID

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m -s /bin/bash $USERNAME 

WORKDIR /workspace
ENV TZ=Asia/Tokyo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
ENV LANG=ja_JP.UTF-8
ENV LC_ALL=ja_JP.UTF-8

## pythonのver指定
ARG PYTHON_VER
ENV PYTHON_VER=$PYTHON_VER

## conda環境の作成
RUN conda create -n myenv python=${PYTHON_VER} -y && \
    conda clean -a

## fastpとkallistoをbiocondaチャネルからインストール
## 250315 kallistoのver指定でエラーが出るため一旦指定なし、今後検討する
RUN conda install -n myenv -c bioconda -y fastp && \
    conda install -n myenv -c bioconda kallisto -y 
