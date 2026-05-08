```shell
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH
export CLASS_PATH=$JAVA_HOME/lib

~/dev/apache-maven-3.9.11/bin/mvn package
~/dev/apache-maven-3.9.11/bin/mvn install
cd core
~/dev/apache-maven-3.9.11/bin/mvn package

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash
nvm install 22
npm config set registry https://registry.npmmirror.com/
npm install
npm run build:distributed

curl -sSL https://dataease.oss-cn-hangzhou.aliyuncs.com/quick_start_v2.sh | bash

# 用户名: admin
# 密码: DataEase@123456
sudo su
bash quick_start_v2.sh

docker ps
CONTAINER ID   IMAGE                                                         COMMAND                  CREATED             STATUS                       PORTS
                   NAMES
1110d7e666ad   registry.cn-qingdao.aliyuncs.com/dataease/dataease:v2.10.22   "/deployments/run-ja…"   About an hour ago   Up About an hour (healthy)   0.0.0.0:8100->8100/tcp, [::]:8100->8100/tcp   dataease
051d4cb9e12c   registry.cn-qingdao.aliyuncs.com/dataease/mysql:8.4.5         "docker-entrypoint.s…"   About an hour ago   Up About an hour (healthy)   3306/tcp, 33060/tcp
                   mysql-de
```