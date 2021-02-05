# Django 入门

原地址[https://www.zmrenwu.com/courses/hellodjango-blog-tutorial/](https://www.zmrenwu.com/courses/hellodjango-blog-tutorial/)

## 环境

### 1.pipenv 管理环境

pycharm创建pipenv管理的名为HelloDjango-blog-tutorial的项目目录
终端输入 set PIPENV_VERBOSITY=-1
给pipenv换国内源，修改pipfile

```
[[source]]
url = "https://pypi.tuna.tsinghua.edu.cn/simple/"
verify_ssl = true
name = "pypi"

[packages]
django = "==2.2.3"

[dev-packages]

[requires]
python_version = "3.8"

```

### 2.安装Django

```
$ pipenv install django==2.2.3
$ pipenv run django-admin startproject blogproject C:\Projects\PythonProjects\HelloDjango-blog-tutorial
$ pipenv run python manage.py runserver
```

默认语言改成中文

## 建立应用

创建blog应用
```
$ pipenv run python manage.py startapp blog
```

再注册blog应用

## 创建 Django 博客的数据库模型

```
# blog/models.py

```

```
$ pipenv run python manage.py makemigrations

django 在 blog 应用的 migrations 目录下生成了一个 0001_initial.py 文件，这个文件是 django 用来记录我们对模型做了哪些修改的文件。

$ pipenv run python manage.py migrate

django 通过检测应用中 migrations 目录下的文件，得知我们对数据库做了哪些操作，然后它把这些操作翻译成数据库操作语言，从而把这些操作作用于真正的数据库。

$ pipenv run python manage.py sqlmigrate blog 0001
你将看到输出了经 django 翻译后的数据库表创建语句，这有助于你理解 django ORM 的工作机制。

$ pipenv run python manage.py createsuperuser
运行 python manage.py createsuperuser 开始创建用户，之后会提示你输入用户名、邮箱、密码和确认密码，按照提示输入即可。
```

## Hello 视图函数 举例子学习一下Django是如何运行的
[https://www.zmrenwu.com/courses/hellodjango-blog-tutorial/materials/63/](https://www.zmrenwu.com/courses/hellodjango-blog-tutorial/materials/63/)
绑定 URL 与视图函数
编写视图函数
配置项目 URL

## 使用 django 模板系统
排序依据的字段是 created_time，即文章的创建时间。- 号表示逆序，如果不加 - 则是正序。

## 定制后台
注册模型
汉化后台
列表显示更多信息
简化表单

## 设计文章详情页的 URL
path('posts/<int:pk>/', views.detail, name='detail')
def get_absolute_url(self):
        return reverse('blog:detail', kwargs={'pk': self.pk})

## 模板继承
## markdown
### 让博客支持 Markdown 语法和代码高亮
### 自动生成目录
### 自动生成摘要

## 自定义侧边栏

## 独立应用 评论功能
细读
表单提交



# django REST framework 教程
[https://www.zmrenwu.com/courses/django-rest-framework-tutorial/](https://www.zmrenwu.com/courses/django-rest-framework-tutorial/)



$ pipenv install djangorestframework django-filter