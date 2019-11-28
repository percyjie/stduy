# stduy
## 1.服务器发布脚本

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

