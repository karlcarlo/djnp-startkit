Vehicle Export Platform API 整车出口平台接口服务端
Django-ninja + PostgreSQL 项目构建流程



#### 创建项目目录和虚拟环境

```bash
mkdir vep_api_djnp
cd vep_api_djnp

python -m venv env
# mac and linux
source env/bin/activate 
# windows
env/bin/activate.bat

```

> (env) jone@localhost:~/workspace/vep_api_djnp$
>
> 命令行提示开头有(env)形式的标识，证明已经进入虚拟环境模式



#### 安装Django 和 Django-Ninja框架

```bash
# Django 4.2
pip install django==4.2.11

# django-ninja 1.0.1
pip install django-ninja==1.1.0

```



#### 在当前目录创建Django项目

```bash
django-admin startproject VehicleExport .
```



#### 创建项目说明文件

```
vi README.md
```



#### 引入`ninja`包

编辑`VehicleExport/settings.py`

```python
INSTALLED_APPS = [
    'ninja', # 加载 ninja
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

```



#### 修改`VehicleExport/settings.py` 文件，进行本地化设置

```python
# 修改使用中文界面
LANGUAGE_CODE = 'zh-Hans'

# 修改时区
TIME_ZONE = 'Asia/Shanghai'

```



#### Web访问测试

我们来创建一个模型的API，在和`VehicleExport/urls.py`文件的相同目录中创建`api.py`文件

新建`VehicleExport/api.py`

```python
from ninja import NinjaAPI

api = NinjaAPI()


@api.get('/hello')
def hello(request):
    return 'Hello'

```

编辑`VehicleExport/urls.py`

```python
from django.contrib import admin
from django.urls import path

from .api import api

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', api.urls),
]

```

##### 启动服务

```bash
# 默认端口 8000
python manage.py runserver

# 或者指定服务端口 http://127.0.0.1:8010
python manage.py runserver 8010

```

在浏览器访问：http://127.0.0.1:8000/api/hello 返回`Hello`字符串

或者访问 http://127.0.0.1:8000/api/docs 返回 `OpenAPI 3.0`格式的接口文档



#### 安装postgres数据库

```bash
pip install psycopg2
```

>  Error: pg_config executable not found.
>
>  可以尝试 `pip install psycopg2-binary`



##### 设置postgres数据库

首先需要使用`Postgres数据库`的客户端工具创建`vehcile_export`库，

然后注释掉包含 `django.db.backends.sqlite3`的`DATABASES`变量默认的全部代码，然后在添加如下代码

```python
# DATABASES = {
#     'default': {
#         'ENGINE': 'django.db.backends.sqlite3',
#         'NAME': BASE_DIR / 'db.sqlite3',
#     }
# }

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'vehicle_export',
        'USER': 'dblink',
        'PASSWORD': 'lk123456',
        'HOST': 'localhost',
        'PORT': 5432,
    }
}
```

> 我这里已经在`postgresql`中配置好了postgresql数据库，用户名密码为`dblink`和`lk123456`，端口为`5432`，所以这里直接使用这些配置，当然你也可以自己创建数据库，然后修改配置

启动服务测试数据库连接是否正常

```bash
python manage.py runserver
```

> 如果正常启动，仅提示类似 `You have 18 unapplied migration(s).`字样，这是因为我们还没有初始化Django后台的数据库表，下面我们就进行数据库的初始化。
>
> 如果提示 `FATAL:  database "vehicle_export" does not exist`，则是因为我们还没有创建数据库或用户验证有误



##### 更新数据迁移

```bash
python manage.py makemigrations # 当数据变更后，创建数据迁移脚本

python manage.py migrate # 执行数据迁移
```

##### 再次启动服务测试数据库连接是否正常

```
python manage.py runserver
```

浏览器访问：http://127.0.0.1:8000/admin 正常进入后台管理登录界面，但是我们还没创建`管理员用户`



#### 创建超级用户

```bash
python manage.py createsuperuser
```

> root
>
> qaz123
>
> 如果提示密码长度太短或太常见，可以按`y`继续创建



#### 安装SimpleUI

> `SimpleUI`基于`Element-UI + Vue`开发

```bash
pip install django-simpleui

# 升级
pip install django-simpleui --upgrade

```

> 如果出现 `error:  invalid command 'bdist_wheel'` 报错，是因为缺少`wheel`库
>
> 我们可以使用`pip`命令来安装`wheel`库
>
> `pip install wheel`
>
> 删除已安装的`simpleui`包
>
> `pip uninstall django-simpleui`
>
> 再次安装`simpleui`不再报错
>
> `pip install django-simpleui`



更新`VehicleModel/settings.py`

```python
# Application definition
INSTALLED_APPS = [
    'ninja',
    'simpleui',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```



##### 启动服务测试`admin`界面是否更新

```
python manage.py runserver
```

浏览器访问：http://127.0.0.1:8000/admin 登录后进入`SimpleUI`样式的`admin`后台管理界面首页

![image-20240408191832996](C:\Users\Jone\iCloudDrive\Markdown\assets\image-20240408191832996.png)

至此我们就完成了项目基本的搭建配置，可以使用`git`初始化并建立一个`init版本`的`tag`

```
# git 初始化
git init

```

在项目根目录新建`.gitignore`，设置`git`忽略文件
内容根据自己的喜好，参考：https://www.gitignore.io/api/django

```
# Created by https://www.toptal.com/developers/gitignore/api/django
# Edit at https://www.toptal.com/developers/gitignore?templates=django

### Django ###
*.log
*.pot
*.pyc
__pycache__/
local_settings.py
db.sqlite3
db.sqlite3-journal
media

# If your build process includes running collectstatic, then you probably don't need or want to include staticfiles/
# in your Git repository. Update and uncomment the following line accordingly.
# <django-project-name>/staticfiles/

### Django.Python Stack ###
# Byte-compiled / optimized / DLL files
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
share/python-wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# PyInstaller
#  Usually these files are written by a python script from a template
#  before PyInstaller builds the exe, so as to inject date/other infos into it.
*.manifest
*.spec

# Installer logs
pip-log.txt
pip-delete-this-directory.txt

# Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/
cover/

# Translations
*.mo

# Django stuff:

# Flask stuff:
instance/
.webassets-cache

# Scrapy stuff:
.scrapy

# Sphinx documentation
docs/_build/

# PyBuilder
.pybuilder/
target/

# Jupyter Notebook
.ipynb_checkpoints

# IPython
profile_default/
ipython_config.py

# pyenv
#   For a library or package, you might want to ignore these files since the code is
#   intended to run in multiple environments; otherwise, check them in:
# .python-version

# pipenv
#   According to pypa/pipenv#598, it is recommended to include Pipfile.lock in version control.
#   However, in case of collaboration, if having platform-specific dependencies or dependencies
#   having no cross-platform support, pipenv may install dependencies that don't work, or not
#   install all needed dependencies.
#Pipfile.lock

# poetry
#   Similar to Pipfile.lock, it is generally recommended to include poetry.lock in version control.
#   This is especially recommended for binary packages to ensure reproducibility, and is more
#   commonly ignored for libraries.
#   https://python-poetry.org/docs/basic-usage/#commit-your-poetrylock-file-to-version-control
#poetry.lock

# pdm
#   Similar to Pipfile.lock, it is generally recommended to include pdm.lock in version control.
#pdm.lock
#   pdm stores project-wide configurations in .pdm.toml, but it is recommended to not include it
#   in version control.
#   https://pdm.fming.dev/#use-with-ide
.pdm.toml

# PEP 582; used by e.g. github.com/David-OConnor/pyflow and github.com/pdm-project/pdm
__pypackages__/

# Celery stuff
celerybeat-schedule
celerybeat.pid

# SageMath parsed files
*.sage.py

# Environments
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# Spyder project settings
.spyderproject
.spyproject

# Rope project settings
.ropeproject

# mkdocs documentation
/site

# mypy
.mypy_cache/
.dmypy.json
dmypy.json

# Pyre type checker
.pyre/

# pytype static type analyzer
.pytype/

# Cython debug symbols
cython_debug/

# PyCharm
#  JetBrains specific template is maintained in a separate JetBrains.gitignore that can
#  be found at https://github.com/github/gitignore/blob/main/Global/JetBrains.gitignore
#  and can be added to the global gitignore or merged into this file.  For a more nuclear
#  option (not recommended) you can uncomment the following to ignore the entire idea folder.
#.idea/

# End of https://www.toptal.com/developers/gitignore/api/django

```

提交全部文件

```bash
# 暂存所有文件
git add .

# 本地提交版本
git commit -m 'base'

# 创建tag
git tag 'base'
```

初始版本创建完成，接下来就可以添加应用了