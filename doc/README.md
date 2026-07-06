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

sudo docker compose up -d

mysql -h127.0.0.1 -uroot -p -P 3316
Password123@mysql

admin
DataEase@123456

sudo docker build -t yiluxiangbei/tortoise .

git pull
cd core
mvn clean package
cd ..
sudo docker build -t yiluxiangbei/tortoise .
cd docker
sudo docker compose up -d
sudo docker compose logs -f

claude
/init
将claude.md改为中文
http://139.199.23.58:8166/de2api/user/modifyPwd
Request Method
POST
Status Code
404 Not Found
```