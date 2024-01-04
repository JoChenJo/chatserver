#  chatserver
可以工作在nginx tcp负载均衡环境中的集群聊天服务器和客户端源码。基于muduo库实现,另外还使用到json库和mysql,redis数据库

编译方式
cd build  
rm -rf *  
cmake ..  
make  
需要用到的库下载链接  
https://github.com/nlohmann/json  
https://github.com/chenshuo/muduo  

还需要安装mysql和redis、hiredis
MYSQL创建数据库CHAT
进入数据库创建以下数据表  
CREATE TABLE User(
id INT PRIMARY KEY AUTO_INCREMENT,
name VARCHAR(50) NOT NULL UNIQUE,
password VARCHAR(50) NOT NULL,
state ENUM('online','offline') DEFAULT 'offline'
);  

CREATE TABLE Friend(
userid INT NOT NULL,  
friendid INT NOT NULL,  
PRIMARY KEY (userid, friendid) 
);  

CREATE TABLE AllGroup(
id INT PRIMARY KEY AUTO_INCREMENT,
groupname VARCHAR(50) NOT NULL ,
groupdesc VARCHAR(200) DEFAULT ''
);  

CREATE TABLE GroupUser(
groupid INT PRIMARY KEY,
userid INT NOT NULL ,
grouprole ENUM('creator','normal') DEFAULT 'normal'
);  

CREATE TABLE OfflineMessage(
userid INT NOT NULL,
groupname VARCHAR(500) NOT NULL
);  

需要安装nginx  TCP负载均衡  
找到nginx安装目录下/conf/nginx.conf  
添加stream {
    upstream MyServer {
        server 127.0.0.1:6000 weight=1 max_fails=3 fail_timeout=30s;
        server 127.0.0.1:6002 weight=1 max_fails=3 fail_timeout=30s;
        }

        server {
            proxy_connect_timeout 1s;
            #proxy_timeout 3s;
            listen 8000;
            proxy_pass MyServer;
            tcp_nodelay on;
        }
}
