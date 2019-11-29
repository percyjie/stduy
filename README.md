# study
### 1.服务器发布脚本

```sh
#!/bin/sh                              

if [ ! -n "$1" ] ;then

    echo "你没有输入分支号"

    exit

else

    echo "本次拉取的分支号是 $1"

fi                                                                                

                                                                                                                       
echo "  ====开始拉取仓库最新代码==== " 

cd /usr/local/xxx;pwd; 


git checkout $1 ;git pull;git status;

echo "         " 

git log --pretty=format:"%h - %an, %ar : %s" -5;

echo "  ====服务器打包===="
# mvn install -Dmaven.test.skip=true;

mvn clean install -Dmaven.test.skip=true;

echo "  =====停止Java应用  等待5秒======"

jps | grep Bootstrap | awk '{print $1;}' | xargs kill -9

#等待5秒

sleep 5 

echo "  ====移动jar包并改名====" 

cd /usr/local/xxx/webapps

rm -rf ROOT.war

cp /usr/local/xxx/web/target/xxx.war /usr/local/tomcat/webapps/;

mv xxx.war ROOT.war;



echo "  =====启动Java应用======"
nohup /usr/local/tomcat/bin/startup.sh & 

                                                                                                                       

#查看日志               
echo "  ===查看日志====";
tail -20f /tmp/logs/xxx.log;

```



### 2.**什么是微服务？**

1.微服务(Microservices Architecture)是一种架构风格，一个大型复杂软件应用由一个或多个微服务组成。系统中的各个微服务可被独立部署，各个微服务之间是松耦合的。每个微服务仅关注于完成一件任务并很好地完成该任务。在所有情况下，每个任务代表着一个小的业务能力。

2.微服务是指开发一个单个 小型的但有业务功能的服务，每个服务都有自己的处理和轻量通讯机制，可以部署在单个或多个服务器上。

3.微服务也指一种种松耦合的、有一定的有界上下文的面向服务架构。也就是说，如果每个服务都要同时修改，那么它们就不是微服务，因为它们紧耦合在一起；如果你需要掌握一个服务太多的上下文场景使用条件，那么它就是一个有上下文边界的服务。

**微服务的优点？**

1.每个微服务都很小，这样能聚焦一个指定的业务功能或业务需求。

2.微服务能够被小团队单独开发，这个小团队是2到5人的开发人员组成。

3.微服务是松耦合的，是有功能意义的服务，无论是在开发阶段或部署阶段都是独立的。

4.微服务能使用不同的语言开发。

5.微服务易于被一个开发人员理解，修改和维护，这样小团队能够更关注自己的工作成果。无需通过合作才能体现价值。

6.微服务允许你利用融合最新技术。

7.微服务只是业务逻辑的代码，不会和HTML,CSS 或其他界面组件混合。

**微服务架构的缺点？**

1.微服务架构可能带来过多的操作。

2.需要DevOps技巧 (http://en.wikipedia.org/wiki/DevOps)。

3.可能双倍的努力。

4.分布式系统可能复杂难以管理。

5.因为分布部署跟踪问题难。

6.当服务数量增加，管理复杂性增加。