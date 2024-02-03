### 1，常见命令

清华源和相关的帮助文档：

~~~
https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/
~~~

* 创建环境：

```python
conda create --name python39 python=3.9
```

注：后加-y可以不用，输入确认创建的y

```text
conda create -n envname python=3.4 scipy=0.15.0 astroib numpy #创建多个包的环境
```

* 激活环境：

```text
conda activate python39
```

注：激活后最前面括号内的名字为环境名

* 退出当前环境：

```text
conda deactivate
```

* 查看已创建环境：

```text
conda info --env
conda info -e
```

注：显示中带*的环境为当前环境

* 删除环境：

```text
conda remove -n python39 --all
```

注：后加-y可以不用，输入确认创建的y

* 查看当前环境安装的包:

```text
conda list   ##获取当前环境中已安装的包
conda list -n python39   ##获取指定环境中已安装的包
```

* 导出当前环境中的包并按照该文件创建新环境：

```text
conda list --explicit > requirements.txt
conda create --name newenv --requirements.txt
```

向一个已存在的环境里安装包

```text
conda install --name newenv --file requirements.txt
```

* 删除包：

```text
conda remove scrapy 
```

删除指定环境中的包

```text
conda remove -n python39 scrapy
```

* 更新包

在当前环境中更新包

```text
conda update scrapy
```

在指定环境中更新包

```text
conda update -n python36 scrapy 
```

更新当前环境所有包

```text
conda update --all
```

* 分享环境

```text
conda env export > environment.yml
```

将该文件放在工作目录下，可以通过以下命令从该文件创建环境

```text
conda env create -f environment.yml
```

* 查找环境中的包

```text
conda search py #模糊查找，只要含py字符串的包名就能匹配到
```

全名查找包，--full-name表示精确查找，即完全匹配名为python的包

```text
conda search --full-name python
```

* 克隆一个环境

```text
# clone_env 代指克隆得到的新环境的名称
# envname 代指被克隆的环境的名称
conda create --name clone_env --clone envname
```

* 更新conda至最新版本 

```text
conda update conda
```

* 查看conda环境管理命令帮助信息

```text
conda create --help
conda -h  #查看帮助信息
```

### 2，安装 pytorch 注意

```python3
docker pull nvidia/cuda:11.7.1-cudnn8-devel-ubuntu22.04
```

首先根据 pytorch 的版本 确定 cuda 的版本，然后在获取对应英伟达预安装dcuda 的镜像。

常见英伟达镜像网址

~~~
https://www.cnblogs.com/chentiao/p/17408994.html
~~~

