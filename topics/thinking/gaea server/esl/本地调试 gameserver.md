# 本地调试 gameserver

1. 使用管理员模式启动 vscode（避免连接消息队列服务器时有报错：Metrics init failed: System.Net.HttpListenerException ）

2. 参照 [1.11 用vscode打开gameserver工程](http://192.168.2.76:8090/pages/viewpage.action?pageId=19238338) 引入工程

3. 看 dotnetcore 文件夹下有一个 config.json 的文件，这里配置了 gameserver 运行需要的参数

   ```
   {
     "rabbitConfig":{
       "hosts":["虚拟机ip:5675","虚拟机ip:5673","虚拟机ip:5674"],
       "port":5672,
       "ports":[5672],
       "username":"guest",
       "password":"guest"
     },
     "assetDir":"..\\game\\Countdown\\Assets\\",
     "exposedIp":"本机ip",
     "exposedPort":32000,
     "listeningPort":32000
   }
   ```

   这里主要的修改内容有：

   1. 修改两个 ip 地址，rabbitConfig 项中 hosts 的虚拟机 ip 和 exposedIp 两个 ip 地址
   2. 修改 assetDir 的目录为正确的资源目录

4. 打开调试设置界面，选中 .NET Core Launch (console)(dotnetcore)
   ![dotnetcore_debug](https://raw.githubusercontent.com/3rdyeah/3rdpics/master/picbed/dotnetcore_debug.png)

5. 点击旁边的齿轮按钮，编辑调试的配置文件 lunch.json

   ```
   {
      // Use IntelliSense to find out which attributes exist for C# debugging
      // Use hover for the description of the existing attributes
      // For further information visit https://github.com/OmniSharp/omnisharp-vscode/blob/master/debugger-launchjson.md
      "version": "0.2.0",
      "configurations": [
           {
               "name": ".NET Core Launch (console)",
               "type": "coreclr",
               "request": "launch",
               "preLaunchTask": "build",
               // If you have changed target frameworks, make sure to update the program path.
               "program": "${workspaceFolder}/bin/Debug/netcoreapp2.0/simulation.dll",
               "args": [
                   "${workspaceFolder}/config.json"
               ],
               "cwd": "${workspaceFolder}",
               // For more information about the 'console' field, see https://aka.ms/VSCode-CS-LaunchJson-Console
               "console": "internalConsole",
               "stopAtEntry": false
           },
           {
               "name": ".NET Core Attach",
               "type": "coreclr",
               "request": "attach",
               "processId": "${command:pickProcess}"
           }
       ]
   }
   ```

   这里在 args 里面加入你要使用的配置文件 "${workspaceFolder}/config.json"

6. 然后即可点击调试按钮启动服务器，记得启动本地 gameserver 前，把虚拟机的 gameserver 容器停掉
   使用 docker ps -a | grep gameserver 找到 gameserver 的 containerid
   使用 docker stop containerid 来停掉 gameserver 容器