# Winform

## 介绍
.Net开发平台中对Windows Form的一种称谓。

## 目录
 - [捕获窗体收到的消息](#捕获窗体收到的消息)
 - [跨线程修改UI]()


## 内容
### 捕获窗体收到的消息

在Form类里面重写WndProc方法
```
 protected override void WndProc(ref Message m)
{ 
	//按理可以收到所有消息，但是我没试过	
	int WM_SYSCOMMAND = 0x112;
	int SC_MINIMIZE = 0xF020;
	int SC_CLOSE = 0xF060;
	if (m.Msg == WM_SYSCOMMAND)
	{
		if (m.WParam.ToInt32() == SC_MINIMIZE) //最小化
		 {
		}
		//关闭窗口
		if (m.WParam.ToInt32() == SC_CLOSE)
		{
		}
	}
            //调用父类的方法
            base.WndProc(ref m);
}
```

### 跨线程修改UI

实现的方式有几种，我用到的是：Task+MethodInvoke

```
Task task = new Task(() =>
            {
                MethodInvoker methodInvoker = new MethodInvoker(() =>
                {
                    SelectedDevice.Text = info;
                });
                this.BeginInvoke(methodInvoker);
            });
            task.Start();
```

