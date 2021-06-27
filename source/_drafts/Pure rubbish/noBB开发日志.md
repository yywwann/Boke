# noBB开发日志

## 2021-02-03 144944

![image-20210203144948544](C:\Users\rxyhc\AppData\Roaming\Typora\typora-user-images\image-20210203144948544.png)



日常起床看一眼数据，直接傻眼了，三条本来应该重叠的线开始飘了

赶紧看一眼nginx报告，全是504错误

![image-20210203162036331](C:\Users\rxyhc\AppData\Roaming\Typora\typora-user-images\image-20210203162036331.png)

>  2021/02/02 17:16:24 **[error]** 7#7: *25602 upstream timed out (110: Connection timed out) while reading response header from upstream, client: 112.32.126.100, server: 42.192.50.232, request: "PUT /api/cdkeylogs/2249/ HTTP/1.1", upstream: "http://172.20.0.2:8000/api/cdkeylogs/2249/", host: "42.192.50.232"

开始百度为什么会出现nginx 504 (110: Connection timed out) 错误





出现这样情况一般是程序执行时间过长，超出Nginx设置的执行时间。

优化方案参考

1.程序逻辑代码优化，减少时间消耗。

2.根据服务器硬件配置加大服务时间设置

nginx配置php执行时间:

  fastcgi_buffers 8 128k;
  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;

nginx配置proxy(反向代理)执行时间:

  proxy_ignore_client_abort on;   # 告诉nginx不要主动关闭连接
  proxy_method POST/GET;
  proxy_connect_timeout 600;
  proxy_read_timeout 600;
  proxy_send_timeout 600;
  proxy_buffer_size 32k;
  proxy_buffers 4 64k;
  proxy_busy_buffers_size 128k;
  proxy_temp_file_write_size 512k;



