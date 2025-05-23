https://xz.aliyun.com/news/12664

文章里边 既有上游也有下游

比如mit 7070端口是burp的下游  burp是mit的上游  mit 9090端口是burp的上游  burp又是mit 9090下游

重定向只需要设置一个上游  mit 7070设置系统代理   burp设置为mit 7070的上游， 代码中使用命令行启动就可以打断点进行调试了

命令行启动： mitmproxy -s mitmproxy_reveser.py --listen-port 7070  --mode upstream:http://127.0.0.1:8080 --ssl-insecure

pyyhon代码启动：


#### 启动 mitmdump
mitmdump(['-s', 'mitmproxy_reveser.py','-p', str(7070), '--mode', "upstream:http://127.0.0.1:8080","--ssl-insecure"])

设置系统代理 让流量经过mitmdump

启动 burp的代理端口默认是8080

开启burp的捕获

完美捕获mitmdump的重定向流量
