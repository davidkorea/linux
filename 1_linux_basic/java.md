1. Download tar.gz format  [openjdk-13.0.1_linux-x64_bin.tar.gz ](https://download.java.net/java/GA/jdk13.0.1/cec27d702aa74d5a8630c65ae61e4305/9/GPL/openjdk-13.0.1_linux-x64_bin.tar.gz) from http://jdk.java.net/13/ 

2. go to `/usr/local`, and `cp` to this path

3. `tar xvf openjdk-13.0.1_linux-x64_bin.tar.gz`, and can get a folder `jdk-13.0.1`

4. `mv jdk-13.0.1/ java`, rename the unpacked folder name to `java` for easy env path

5. edd env path to the tail of `/etc/profile`
  ```
  79 export JAVA_HOME=/usr/local/java
  80 export PATH=$PATH:$JAVA_HOME/bin
  81 export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
  82 export JRE_HOME=$JAVA_HOME/jre
  ```
6. `source /etc/profile`

7. `java -version`
