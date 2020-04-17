##使用 Django

使用 `Docker Compose` 配置并运行一个 `Django/PostgreSQL` 应用。

### 1、编辑 Dockerfile

```bash
FROM python:3
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
COPY requirements.txt /code/
RUN pip install -r requirements.txt
COPY . /code/
```

以上内容指定应用将使用安装了 Python 以及必要依赖包的镜像



### 2、准备requirements.txt

在 `requirements.txt` 文件里面写明需要安装的具体依赖包名。

```bash
Django>=2.0,<3.0
psycopg2>=2.7,<3.0
```



### 3、编辑docker-compose.yml

`docker-compose.yml` 文件将把所有的东西关联起来。它描述了应用的构成（一个 web 服务和一个数据库）、使用的 Docker 镜像、镜像之间的连接、挂载到容器的卷，以及服务开放的端口

```bash
version: "3"
services:

  db:
    image: postgres

  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    links:
      - db
```

现在我们就可以使用 `docker-compose run` 命令启动一个 `Django` 应用了。

```bash
[root@docker data]# docker-compose run web django-admin startproject django_example .
```

由于 web 服务所使用的镜像并不存在，所以 Compose 会首先使用 `Dockerfile` 为 web 服务构建一个镜像，接着使用这个镜像在容器里运行 `django-admin startproject django_example` 指令。

这将在当前目录生成一个 `Django` 应用

```bash
$ ls
Dockerfile       docker-compose.yml          django_example       manage.py       requirements.txt
```

首先，我们要为应用设置好数据库的连接信息。用以下内容替换 `django_example/settings.py` 文件中 `DATABASES = ...` 定义的节点内容。

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'HOST': 'db',
        'PORT': 5432,
    }
}
```

这些信息是在 [postgres](https://hub.docker.com/_/postgres/) 镜像固定设置好的。然后，运行 `docker-compose up` ：

```bash
[root@docker data]# docker-compose up              
Starting data_db_1 ... done
Starting data_web_1 ... done
Attaching to data_db_1, data_web_1
db_1   | Error: Database is uninitialized and superuser password is not specified.
db_1   |        You must specify POSTGRES_PASSWORD to a non-empty value for the
db_1   |        superuser. For example, "-e POSTGRES_PASSWORD=password" on "docker run".
db_1   | 
db_1   |        You may also use "POSTGRES_HOST_AUTH_METHOD=trust" to allow all
db_1   |        connections without a password. This is *not* recommended.
db_1   | 
db_1   |        See PostgreSQL documentation about "trust":
db_1   |        https://www.postgresql.org/docs/current/auth-trust.html
data_db_1 exited with code 1
web_1  | Watching for file changes with StatReloader
web_1  | Performing system checks...
web_1  | 
web_1  | System check identified no issues (0 silenced).
web_1  | 
web_1  | You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
web_1  | Run 'python manage.py migrate' to apply them.
web_1  | March 31, 2020 - 05:56:35
web_1  | Django version 2.2.11, using settings 'django_example.settings'
web_1  | Starting development server at http://0.0.0.0:8000/
web_1  | Quit the server with CONTROL-C.
web_1  | [31/Mar/2020 05:56:39] "GET / HTTP/1.1" 200 16348
web_1  | [31/Mar/2020 05:56:39] "GET /static/admin/css/fonts.css HTTP/1.1" 200 423
web_1  | [31/Mar/2020 05:56:39] "GET /static/admin/fonts/Roboto-Bold-webfont.woff HTTP/1.1" 200 86184
web_1  | [31/Mar/2020 05:56:39] "GET /static/admin/fonts/Roboto-Regular-webfont.woff HTTP/1.1" 200 85876
web_1  | [31/Mar/2020 05:56:40] "GET /static/admin/fonts/Roboto-Light-webfont.woff HTTP/1.1" 200 85692
web_1  | Not Found: /favicon.ico
web_1  | [31/Mar/2020 05:56:41] "GET /favicon.ico HTTP/1.1" 404 1986
```

这个 `Django` 应用已经开始在你的 Docker 守护进程里监听着 `8000` 端口了。打开 `127.0.0.1:8000` 即可看到 `Django` 欢迎页面。

你还可以在 Docker 上运行其它的管理命令，例如对于同步数据库结构这种事，在运行完 `docker-compose up` 后，在另外一个终端进入文件夹运行以下命令即可：

```bash
[root@docker data]# docker-compose run web python manage.py syncdb
```