FROM debian:buster-slim

MAINTAINER ziyear <SuperZiyear@163.com> 

# 将dockerfile上下文目录中的nginx-1.18.1.tar.gz文件拷贝进容器根目录并解压
ADD nginx-1.18.0.tar.gz /usr/local/src/nginx/
ADD jre-8u301-linux-x64.tar.gz /usr/local/java/

# 配置JAVA环境变量
ENV JAVA_HOME /usr/local/java/jre1.8.0_301
ENV CLASSPATH .:${JAVA_HOME}/lib
ENV PATH ${JAVA_HOME}/bin:$PATH


# 切换至nginx目录
WORKDIR /usr/local/src/nginx/nginx-1.18.0

# 配置时区
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \ 
    echo 'Asia/Shanghai' >/etc/timezone && \
	# 先备份原源配置文件然后将debian的apt源改成中科大源
	cp /etc/apt/sources.list /etc/apt/sources.list.bak && \
	sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list && \
	sed -i 's|security.debian.org/debian-security|mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list && \
	# 初始版的nginx需要的依赖：gcc zlib1g zlib1g-dev openssl libssl-dev libpcre3 libpcre3-dev
	# libgd-dev和libexif-dev需要的依赖
	apt-get clean; apt-get update -y; apt-get install -y;\
	apt-get install --assume-yes apt-utils;\
	apt-get -y install gcc;\
	apt-get -y install zlib1g zlib1g-dev;\
	apt-get -y install openssl libssl-dev;\
	apt-get -y install libpcre3 libpcre3-dev;\
	apt-get -y install make && \
	# 创建nginx用户，yum方式安装无需做此步骤，因为默认yum安装会自动创建nginx用户
	# 咱们下面命令以nginx用户运行，因此我们需要在容器中创建nginx用户
	# -M(不创建主目录) -s(不允许登录)
	useradd -M -s /sbin/nologin nginx && \
	# 编译安装nginx
	# --with-http_stub_status_module -> Nginx性能统计模块
	# --with-http_ssl_module -> ssl加密模块
	# --add-module=/ngx_http_image_filter_module -> 图片裁剪模块
	./configure \
	--prefix=/usr/local/nginx \
	--user=nginx \
	--group=nginx \
	--sbin-path=/usr/local/nginx/sbin/nginx \
	--modules-path=/usr/local/nginx/modules \
	--conf-path=/usr/local/nginx/conf/nginx.conf \
	--error-log-path=/usr/local/nginx/logs/error.log \
	--http-log-path=/usr/local/nginx/logs/access.log \
	--pid-path=/usr/local/nginx/logs/nginx.pid \
	--lock-path=/usr/local/nginx/logs/nginx.lock \
	--with-http_stub_status_module \
	--with-http_ssl_module  && \
	make && make install && \
	mkdir -p /usr/local/nginx/conf.d /usr/local/nginx/cert && \
	rm -rf /usr/local/src/nginx/*
	
# 设置挂在目录 启动容器时便于使用-v本地目录影射到容器目录
VOLUME ["/usr/local/nginx/html/", "/usr/local/nginx/conf/", "/usr/local/nginx/conf.d/", "/usr/local/nginx/logs"]

WORKDIR /

# 定义向外暴露的端口号，多个端口用空格做间隔，启动容器时便于使用-p将此端口向宿主机端口映射
EXPOSE 80/tcp 443/tcp

# 解释 -> '-g' 'daemon off'
# nginx默认是以后台模式启动的，Docker未执行自定义的CMD之前，nginx的pid是1，
# 执行到CMD之后，nginx就在后台运行，bash或sh脚本的pid变成了1，
# 所以一旦执行完自定义CMD，nginx容器也就退出了，为了保持nginx的容器不退出，应该关闭nginx后台运行
CMD ["/usr/local/nginx/sbin/nginx", "-g", "daemon off;"]
