如果SSH服务器支持运行命令(例如ssh user@host"cmd; cmd; ...")，那么exec_command()就足够了。但是某些SSH服务器(例如，诸如交换机，路由器之类的网络设备)仅支持启动交互式会话，那么您需要使用invoke_shell() 它返回一个Channel。

Transport和Channel均为SSH术语(Paramiko除外)
创建一个 Paramiko Transport 对象并连接到远程设备。然后，它打开一个新的 Channel 并调用 invoke_shell() 方法来启用交互式 shell 模式（即用户能够输入和输出命令）。
send_command() 方法将指定的命令发送到远程设备，并等待直到收到一个以交换机提示符结尾的响应。然后，它返回收到的所有响应文本

https://zhuanlan.zhihu.com/p/609450331?utm_id=0
