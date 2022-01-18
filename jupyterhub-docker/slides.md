---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
fonts:
  mono: 'Cascadia Code'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# persist drawings in exports and build
drawings:
  persist: false
---

## CFPS JupyterHub 大作业讲解

# Dockerfile 及周边脚本讲解

#### 山东大学（威海） 2020级 数据科学与人工智能实验班

#### 讲解人：任鹏飞

#### 2022 年 1 月 18 日

幻灯片在 https://github.com/lie-flat/slides 仓库

---
layout: two-cols
---

# 目录结构

```
cfps-jupyterhub/cfps-notebook
├── Dockerfile
├── ipython_config.py
├── requirements.txt
├── start-notebook.sh
└── startup.py
```

::right::

```
cfps-jupyterhub/minimal-notebook-py310
├── Dockerfile
├── fix-permissions
├── jupyter_notebook_config.py
├── LICENSE.md
├── README.md
├── start-notebook.sh
├── start.sh
├── start-singleuser.sh
└── test
    ├── data
    │   ├── Jupyter_logo.svg
    │   ├── notebook_math.ipynb
    │   └── notebook_svg.ipynb
    ├── test_container_options.py
    ├── test_inkscape.py
    ├── test_nbconvert.py
    ├── test_package_managers.py
    ├── test_pandoc.py
    ├── test_python.py
    └── test_start_container.py
```


---

因为 Jupyter 官方没有提供使用了 Python 3.10 的镜像，所以我们将官方的配置文件拷贝了一份，修改一下，作为一个 base image.

文件： `cfps-jupyterhub/minimal-notebook-py310/Dockerfile`
基于： 
- https://github.com/jupyter/docker-stacks/blob/master/base-notebook/Dockerfile
- https://github.com/jupyter/docker-stacks/blob/master/minimal-notebook/Dockerfile

除了合并上面两个 Dockerfile 之外，这个文件我做的主要改动如下


```diff
+LABEL maintainer="Levi Zim <levi_zim@outlook.com>"
-LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
+ARG PYTHON_VERSION=3.10.1
-ARG PYTHON_VERSION=default
+ARG CONDA_MIRROR=https://mirrors.tuna.tsinghua.edu.cn/github-release/conda-forge/miniforge/LatestRelease
-ARG CONDA_MIRROR=https://github.com/conda-forge/miniforge/releases/latest/download
```

- 把 maintainer 改为我自己
- 把 python 版本改为 3.10.1
- 修改 conda 镜像网址，换成国内源以加速下载
- 这个镜像我发到了 docker hub 上，你可以通过 `rsworktech/minimal-notebook-py310` 这个名称来拉取

---

然后是我们的 Dockerfile: `cfps-jupyterhub/cfps-notebook/Dockerfile`

基于官方的这个 Dockerfile: https://github.com/jupyter/docker-stacks/blob/master/scipy-notebook/Dockerfile

除去官方的那些，我编写的大致如下：

<div style="height: 20ch;overflow: scroll">

```docker
FROM rsworktech/minimal-notebook-py310:latest
LABEL maintainer="Levi Zim <levi_zim@outlook.com>"
USER root
RUN mkdir /etc/ipython
COPY startup.py /opt/startup.py
COPY ipython_config.py /etc/ipython/ipython_config.py

USER ${NB_UID}
RUN mamba install --quiet --yes \
    'jupyterlab-language-pack-zh-CN' && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

COPY requirements.txt /tmp/requirements.txt
RUN pip3 install --no-cache -r /tmp/requirements.txt
RUN mkdir "/home/${NB_USER}/data" && \
    fix-permissions "/home/${NB_USER}/data" && \
    mkdir "/home/${NB_USER}/team-shared" && \
    fix-permissions "/home/${NB_USER}/team-shared"

USER root
COPY start-notebook.sh /usr/local/bin

USER ${NB_UID}
WORKDIR "${HOME}"
```

</div>

- 把我们的 ipython 自定义配置和启动脚本拷贝过去
- 安装中文语言包
- 安装我们从 `requirements.txt` 中指定的包
- 创建好 `～/data` 目录和 `～/team-shared` 目录，修正权限
- 使用我们修改过的 `start-notebook.sh` 替换掉原来的

---

# ipython 设置与启动脚本

```python
c.InteractiveShellApp.exec_files = [
    '/opt/startup.py'
]
```

在 IPython Kernel 启动时，执行 `/opt/startup.py` 脚本：

```python
import sys
sys.path.append("/home/jovyan/data/cfps-analyze/process")
from cfps_shell import *
sys.path.append("/home/jovyan/data")
from cfps_dvapis import *
from pyecharts.globals import CurrentConfig, NotebookType
CurrentConfig.NOTEBOOK_TYPE = NotebookType.JUPYTER_LAB
```

- 首先，把 `/home/jovyan/data/cfps-analyze/process` 路径添加到 Python 的搜索路径中，这样我们就可以把 CFPS Shell 引用进来
- 然后，把 `/home/jovyan/data` 路径添加到 Python 的搜索路径中，这样我们就可以把 cfps_dvapis 引进来
- 然后将 pyecharts 的运行设置修改一下，让它知道它是在 Jupyter Lab 中运行的

---

# requirements.txt

```
mysql-connector-python
PyMySQL
tqdm
pyecharts
jupyterlab_cfps_preload
```

# start-notebook.sh

—

和官方原版文件相比，我们在开头加了一句话,用来在容器启动时执行我们的自定义脚本。

```bash
/home/jovyan/data/on_container_start_up.sh >> /tmp/startup.log 2>&1
```