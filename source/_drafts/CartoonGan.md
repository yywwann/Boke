---
title: CartoonGan
date: 2020-01-08 23:25:24
tags:
- GAN
categories:
- GAN
---

# CartoonGAN

## Environment
> macOS Mojave 10.14.2

First clone this repo:

```bash
git clone https://github.com/mnicnc404/CartoonGan-tensorflow.git
```

<!--more-->
To run code in this repo properly, you will need:
- [Python 3.6](https://www.python.org/downloads/release/python-360/)
- [TensorFlow 2.0 Alpha](https://www.tensorflow.org/alpha)
- [tqdm](https://github.com/tqdm/tqdm)
- [imageio](https://pypi.org/project/imageio/)
- [tb-nightly](https://pypi.org/project/tb-nightly/)

For environment management, we recommend [Conda](https://conda.io/projects/conda/en/latest/user-guide/install/index.html#regular-installation). You can get `conda` by installing [Anaconda](https://www.anaconda.com/distribution/#download-section) or [Miniconda](https://docs.conda.io/en/latest/miniconda.html).
You can install all the packages by running the following commands:

```bash
# Select .yml file depending on your os and whether GPU is available
# environment_linux_gpu.yml for linux with nvidia gpu
# environment_linux_cpu.yml for linux without nvidia gpu
# environment_mac_cpu.yml for mac without nvidia gpu
conda env create -n cartoongan -f environment_linux_gpu.yml
# Installs python==3.6.8 to the new environment
conda activate cartoongan
# to deactivate this env, run "conda deactivate"
```

If Anaconda is not available, you can also run:

```bash
pip install -r requirements_gpu.txt
# use `requirements_cpu` if GPU is not available
```


> mkl一直失败,没办法,只好手动创建Anaconda,然后手动一个个install requirements_cpu.txt 中的包



