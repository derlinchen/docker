1. 首先对springboot项目进行打包：orange-0.0.1-SNAPSHOT.war

2. 创建Dockerfile，并将Dockerfile与项目包放在同一文件夹下

		FROM java:8
		VOLUME /tmp
		ADD orange-0.0.1-SNAPSHOT.war app.jar
		RUN bash -c 'touch /app.jar'
		EXPOSE 8080
		ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]


3. 在文件加下执行docker命令生成images：docker build -t orange:1.0.0 .

4. 运行镜像生成容器：docker run -d -p 8080:8080 orange:1.0.0
