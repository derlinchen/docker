## Docker的使用说明 ##

### 一、镜像 ###

1. 获取镜像：
		
		docker pull repository:tag
		repository-镜像名，tag-版本

2. 查看镜像列表：

		docker images

3. 为镜像添加标签：
	
		git tag ubuntu:16.04 ubuntu:16 
		ubuntu:16.04表示现有的镜像，ubuntu:16表示打过标签后的镜像

4. 查看镜像历史：

	 	docker history ubuntu:16.04

5. 查询镜像：

		docker search ubuntu 
		可通过 `docker search --help`查看`docker search`可有哪些选项

6. 删除镜像：
		
		docker rmi ID
		通过id删除镜像
		docker rmi ubuntu:16.04 
		通过仓库与tag删除镜像
		删除镜像之前要先删除容器

7. 删除所有镜像：
		
		 docker rmi `docker images -q`

8. 基于容器创建镜像：

		docker commit -m 'msg' -a 'author' containtnerid repository:tag
		docker commit -m 'init' -a -'climber' 5c6da3c mylinux:v1

9. 将镜像导出到本地电脑：

		docker save -o filename repository:tag
		docker save -o ubuntu_16.04.tar ubuntu:16.04

10. 从本地电脑将镜像导入到docker:

		docker load -i filename
		docker load -i ubuntu_16.04.tar

11. 上传镜像到dockerhub上：

		1.先在docker中登录dockerhub
		docker login 根据提示输入用户名、密码、邮箱
		
		2.在dockerhub上创建repository

		3.在docker中打tag，tag的名称为：dockerhub用户名\repository的名称

		4.进行推送：docker push dockerhub用户名\reponsitory名称

### 二、容器 ###

1. 创建容器：

		docker create -it ubuntu:16.04

2. 启动容器：

		docker start containerid

3. 进入容器：

		docker attach containerid
		docker exec -it containerid /bin/bash
		exec在Ctrl+D退出后，容器仍然在执行

4. 停止容器：

		docker stop containerid

5. 新建启动容器：

		docker run -it ubuntu:16.04 /bin/bash

6. 暂停容器：

		docker pause containerid

7. 删除容器：

		docker stop containerid	//停止容器
		docker rm containerid	//删除容器

8. 查看所有容器：

		docker ps -a

9. 导出容器：

		docker export -o filename ubuntu:16.04

10. 导入容器：

		docker import filename imagename
		//容器导入docker后会自动生成为镜像

### 三、Docker仓库 ###

1. 搭建私有仓库

		docker run -d -p 5000:5000 -v /opt/data/registry:/var/lib/registry registry:2
	
		// 修改配置文件：
		sudo gedit /etc/default/docker
		// 修改配置文件
		DOCKER_OPTS="--insecure-registry 0.0.0.0:5000"

2. 私有仓库推送镜像

		docket tag java:8 127.0.0.1:5000/test

		docker push 127.0.0.1:5000/test

### 四、数据卷 ###

1. 创建数据卷

		docker volume create -d local test

2. 查看volume路径

		// 查看是因为没有相应的权限
		ls -l /var/lib/docker/volumes

3. 数据卷使用：

		// 在u1容器中定义数据卷
		docker run -it -v /dbdata --name u1 ubuntu
		// 将u1的数据卷挂载到u2
		docker run -it --volumes-from u1 --name u2 ubuntu
		// 将u1的数据卷挂载到u3 在对其中一个容器中的数据卷修改时，都会相应的做出修改
		docker run -it --volumes-from u1 --name u3 ubuntu

4. 对数据卷进行备份：

		// 创建worker容器，将u3下的数据卷放到worker容器下的/backup目录，将数据卷的内容备份到宿主主机的/backup/backup.tar下
		docker run --volumes-from u3 -v $(pwd):/backup --name worker ubuntu tar cvf /backup/backup.tar dbdata

5. 数据卷恢复：

		//创建带有数据卷的容器u4
		docker run -v /dbdata --name u4 ubuntu /bin/bash
		// 创建容器 busybox，将数据卷容器u4挂载到busybox,使用untar解压到所挂载的容器卷中
		docker run --volumes-from u4 -v $(pwd):/backup busybox tar xvf /backup/backup.tar

### 五、端口映射 ###

1. 随机映射端口：
		
		// 使用-P随机生成映射端口
		docker run -d -P training/webapp python app.py

2. 映射所有接口地址：

		docker run -d -p 5000:5000 training/webapp python app.py

3. 映射到指定地址的制定端口：

		docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py 
		
4. 映射到指定地址的任意端口：

		docker run -d -p 127.0.0.1::5000 training/webapp python app.py
		
5. 查看端口所绑定的地址：

		docker port 7b8 5000

6. 容器互联：

		// 创建容器
		docker run -d --name db training/postgres
		// 使用容器互联
		docker run -d -P --name web --link db:db training/webapp python app.py

### 六、Dockerfile ###

1. Dockerfile创建：

		FROM ubuntu:18.04
	
		LABEL maintainer derlin<derlin_nj@163.com>
		
		RUN  apt-get update && apt-get install -y nginx
		
		CMD  /usr/sbin/nginx
		
		// 执行Dockerfile: 
		// 1. 将文件复制到无其他文件的文件夹
		// 2. 执行docker build -t ubuntu:v1 . 
		// 3. ubuntu:v1表示镜像及版本，.表示执行当前的Dockerfile文件

2. docker build指令说明：

		ARG  定义创建镜像过程中使用的变量
		FROM  指定所创建镜像的基础镜像
		LABEL  为生存的镜像添加元数据标签信息
		EXPOSE  声明镜像内服务监听的端口
		ENV  指定环境变抵
		ENTRYPOINT  指定镜像的默认入口命令
		VOLUME  创建一个数据卷挂载点
		USER  指定运行容器时的用户名或UID
		WORKDIR  配置工作目录
		ONBUILD  创建子镜像时指定自动执行的操作指令
		STOPSIGNAL  指定退出的信号值
		HEALTH  CHECK  配置所启动容器如何进行健康检查
		SHELL  指定默认shell类型
		RUN 定义创建镜像过程中使用的变量。
		CMD 启动容器时指定默认执行的命令
		ADD 添加内容到镜像
		COPY 复制内容到镜像

3. 创建镜像：

		执行docker build -t ubuntu:v1 . 

4. 选择父镜像：

		通过FROM命令执行父镜像


