---
title: jupyter-notebook安装及配置
tags: python, jupyter-notebook, 配置
grammar_cjkRuby: true
---


## 1.jupyter-notebook安装
如果使用的是纯python环境，则直接使用pip install 安装对应版本的jupyter-notebook；如果是anaconda安装的python，则内置有jupyter-notebook，直接可以使用。
## 2.jupyter-notebook主题配置
两种主题：[jupyter-themes][1]，[jupyter-themer][2]，直接使用pip安装即可
## 3.jupyter-notebook工具栏配置
安装jupyter-notebook扩展[jupyter_contrib_nbextensions][3]

``` python
pip install jupyter_contrib_nbextensions
conda install -c conda-forge jupyter_contrib_nbextensions
```
两种方式选一种
另外选择安装配置工具[configurator][4]
```python
conda install -c conda-forge jupyter_nbextensions_configurator
pip install jupyter_nbextensions_configurator
两种方式选一种
启动配置：
jupyter nbextensions_configurator enable --user
```
注意：
在安装python程序包的过程中如果出现错误，大概有3个原因：
* 安装包名字拼错
* 安装包与python版本不一致，请使用wheel包安装[pythonlibs][5]
* 网络不稳定，如有必要则翻墙重新安装

  [1]: https://github.com/dunovank/jupyter-themes
  [2]: https://github.com/transcranial/jupyter-themer
  [3]: https://github.com/ipython-contrib/jupyter_contrib_nbextensions
  [4]: https://github.com/Jupyter-contrib/jupyter_nbextensions_configurator
  [5]:http://www.lfd.uci.edu/~gohlke/pythonlibs/