```shell
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH
export CLASS_PATH=$JAVA_HOME/lib

~/dev/apache-maven-3.9.11/bin/mvn package
~/dev/apache-maven-3.9.11/bin/mvn install
cd core
~/dev/apache-maven-3.9.11/bin/mvn package
```