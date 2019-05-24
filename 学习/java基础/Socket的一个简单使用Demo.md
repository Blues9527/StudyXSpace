# Socket
### server端
```
//port 端口号，需与客户端的端口号一致；
ServerSocket ss = new ServerSocket(int port);
Socket socket = ss.accept();

//通过流传输
InputStream is = socket.getInputStream();

//通过字节流传输,初始化大小1024
byte[] bytes = new byte[1024];
int len;//用于判断是否读取完毕

//使用StringBuffer，线程安全
StringBuffer sb = new StringBuffer();

//通过while循环读取流
while((len = is.read(bytes))!=-1){
    sb.append(new String(bytes,0,len,"UTF-8"));//统一编码格式
}
//输出内容
System.out.println(sb);

//使用后需关闭，防止内存溢出
is.close();
socket.close();
ss.close();
```
### client端
    
```
/**
*@params host : ip地址
*@params port : 端口号
*/
//实例化一个socket
Socket socket = new Socket(String host,int port);

//开启输出流
OutputStream os = socket.getOutputStream();

//向服务端socket传数据
socket.getOutputStream().write(new String("hello").getBytes("UTF-8"));//需统一编码格式

//使用完后关闭
os.cloes();
socket.close();
```