---
title: allennlp安装踩坑
date: 2019-05-22 22:09:27
tags:
- allennlp
- 包安装
english_title: allennlp_install
categories:
- install

---

> 安装allennlp的踩坑之路，踩了不少坑最后选择'Installing from source'的安装方法，排坑后下面方法亲测可用

<!-- more -->

## Installing from source

安装步骤：

### 1.下载GitHub文件

```shell
git clone https://github.com/allenai/allennlp.git
```

### 2.创建conda环境

```shell
conda create -n allennlp python=3.6
```

### 3.激活环境下载依赖文件

- 激活环境

  ```shell
  source activate allennlp
  ```

- 进入github上下载的文件夹

- 下载依赖文件

  ```shell
  pip install -r requirements.txt
  ```

  遇到报错问题，参考下一小节，所欲问题解决。

### 4.安装allennlp

```shell
pip install --editable .
```

### 5.测试

```shell
allennlp
```

成功后效果如下：

```shell
$ allennlp
2019-05-22 21:58:42,297 - INFO - pytorch_pretrained_bert.modeling - Better speed can be achieved with apex installed from https://www.github.com/nvidia/apex .
usage: allennlp

Run AllenNLP

optional arguments:
  -h, --help     show this help message and exit
  --version      show program's version number and exit

Commands:

    configure    Run the configuration wizard.
    train        Train a model.
    evaluate     Evaluate the specified model + dataset.
    predict      Use a trained model to make predictions.
    make-vocab   Create a vocabulary.
    elmo         Create word vectors using a pretrained ELMo model.
    fine-tune    Continue training a model on a new dataset.
    dry-run      Create a vocabulary, compute dataset statistics and other
                 training utilities.
    test-install
                 Run the unit tests.
    find-lr      Find a learning rate range.
    print-results
                 Print results from allennlp serialization directories to the
                 console.
```



## 遇到的问题

### 问题1

#### 报错信息：

ERROR: Failed building wheel for jsonnet

![20190522155853293350470.jpg](http://image.nysdy.com/20190522155853293350470.jpg)

#### 解决方法：

```shell
conda install -c conda-forge jsonnet
```

### 问题2

#### 报错信息：

报的都是某些包的版本问题

```shell
ERROR: botocore 1.12.152 has requirement urllib3<1.25,>=1.20; python_version >= "3.4", but you'll have urllib3 1.25.2 which is incompatible.
ERROR: aws-sam-translator 1.11.0 has requirement jsonschema~=2.6, but you'll have jsonschema 3.0.1 which is incompatible.
ERROR: cfn-lint 0.20.3 has requirement jsonschema~=2.6, but you'll have jsonschema 3.0.1 which is incompatible.
ERROR: cfn-lint 0.20.3 has requirement requests<=2.21.0,>=2.15.0, but you'll have requests 2.22.0 which is incompatible
```

#### 解决方法

根据报错信息下载相应安装包即可

## 问题3

#### 报错信息：

```shell
ImportError: dlopen: cannot load any more object with static TLS
___________________________________________________________________________
Contents of /home/minelab/anaconda3/envs/allennlp/lib/python3.6/site-packages/sklearn/__check_build:
__init__.py               setup.py                  _check_build.cpython-36m-x86_64-linux-gnu.so
__pycache__
___________________________________________________________________________
It seems that scikit-learn has not been built correctly.

If you have installed scikit-learn from source, please do not forget
to build the package before using it: run `python setup.py install` or
`make` in the source directory.

If you have used an installer, please check that it is suited for your
Python version, your operating system and your platform.
```

#### 解决方法：

下载更低版本的scikit-learn,例如

```shell
pip install scikit-learn=0.20.3
```

## 参考链接

- https://github.com/pytorch/pytorch/issues/10443
- https://github.com/pypa/pip/issues/4330

# 安装的启示

### 环境问题

- 最基本的就是先去网上查这个错误的解决方法
- 网上的解决不了的，先猜猜大概率是哪方面的问题。
  - 比如大概率是各种版本互相之间不适配的问题，那就调试版本，一般都会告诉你哪个有问题，比如上面的scikit-learn问题。