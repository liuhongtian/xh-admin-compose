# 使用Docker Compose部署若依

## 自定义镜像

为了方便添加依赖，基于ubuntu创建了一个自定义的OpenJDK21的镜像，如果有其他合适的JDK镜像，也可以直接使用。

创建镜像的命令如下：

```bash
docker build . -t hwjdk:21
```

以上命令保存在 **makeimage.sh** 脚本中，位于 **custom-images/jdk** 目录下，请到此目录执行命令。

之后，可以执行如下命令将镜像导出为tar文件，并压缩：

```bash
docker save -o hwjdk21.tar hwjdk:21
gzip hwjdk21.tar
```

这将创建 **hwjdk21.tar.gz** 文件，将其上传到服务器，并执行如下命令加载镜像：

```bash
docker load -i hwjdk21.tar.gz
```

## 目录及文件说明

- compose.yml：服务定义文件。
- mysql：MySQL数据目录。
- ruoyi/java：ruoyi-admin.jar放置到此目录。
- ruoyi/mysql：若依框架的初始化脚本放置到此目录，按字母顺序执行，请自行保证合理的顺序，例如添加序号。
- ruoyi/nginx：ruoyi-ui的打包内容（dist目录下的内容）放置到此目录。
- ruoyi/nginx-conf.d：nginx镜像会加载此目录下的配置文件，目录里的default.conf文件是从nginx镜像里复制出来，并调整了配置修改后的文件，将覆盖镜像中的同名文件。需要的话，可以添加新的配置文件。

## 服务维护

### 服务部署并启动

在本目录下执行如下命令部署并启动服务：

```bash
docker compose up -d
```

执行如下命令查看服务运行状态：

```bash
docker compose ps
```

### 服务停止

```bash
docker compose stop
```

### 服务启动

使用如下命令启动已停止的服务：

```bash
docker compose start
```

### 服务卸载

执行以下命令，将卸载compose.yml中定义的全部服务，并删除容器：

```bash
docker compose down
```

### 查看服务日志

```bash
docker compose logs -f
```

这将查看compose.yml中定义的若有服务的日志，执行命令时附加服务名，可以查看指定服务的日志。例如：

```bash
docker compose logs -f ruoyi-ui
```

以上命令查看 **ruoyi-ui** 的日志。

## 服务定义

compose.yml中定义了RuoYi前后端分离版的所有模块及依赖项，包括：

- mysql：MySQL数据库。
- redis：Redis服务。
- ruoyi-admin：后端。
- ruoyi-ui：前端。

> 服务名相当于同一个compose集群内的各个容器的主机名，不要随意修改。如果修改了，请同时修改相关配置。

### compose.yml

以下是compose.yml的内容：

```yaml
services:
  mysql:
    image: mysql
    ports:
      - "23306:3306"
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: ry
      MYSQL_USER: ruoyi
      MYSQL_PASSWORD: ruoyi123456
    volumes:
      - ./mysql:/var/lib/mysql
      - ./ruoyi/mysql:/docker-entrypoint-initdb.d

  redis:
    image: redis
    ports:
      - "6379"
    restart: always
  
  ruoyi-admin:
    image: hwjdk:21
    ports:
      - "8080"
    restart: always
    volumes:
      - ./ruoyi/java:/jars
      - ./ruoyi/uploadPath:/var/ruoyi/uploadPath
    command: ["-Xmx1024m", "-jar", "/jars/ruoyi-admin.jar"]

  ruoyi-ui:
    image: nginx
    ports:
      - "20080:80"
    restart: always
    volumes:
      - ./ruoyi/nginx-conf.d:/etc/nginx/conf.d
      - ./ruoyi/nginx:/usr/share/nginx/html
```
