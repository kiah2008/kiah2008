修改Hosts解决Github访问难题.

# 1 访问慢、无法加载还是无法通讯？
这三种情况是有区别的，含义不一样：
访问慢：连接延迟高，内容能够被加载但需要较长的时间。
无法加载：浏览器无法打开网址。
无法通讯：无法进行直接的网络通讯，包括了上一种情况。
自然导致这三种情况的原因也不尽相同：

访问慢：服务器或 CDN 节点的地理位置相对较远，难以物理超度。注意这里的表现是延迟高，不一定是每秒传输速度慢。
无法加载：可能由于长时间的未响应，即访问慢的情况，导致浏览器判定无法加载内容；可能由于网址对应内容不能被直接访问，即无可访问内容或无权限访问。
无法通讯：这类情况往往是 IP 解析错误，即遭受 DNS 污染；否则就是 IP 服务器出现了内部错误。
# 2 检测一下
可以利用两个工具来判断不同域名或 IP 地址是上述那种情况。笔者以github.com为例，实际操作一遍检测的过程，看看是什么情况：

首先利用网络上的ping工具，例如[这个](https://ping.chinaz.com/)，检测网址、IP 地址的通讯情况。输入要检测的网址github.com，点击「Ping 检测」。工具提供的服务是利用自己分布在各地的网络节点的本机ping工具，执行对网址的ping操作，汇总结果，统计响应网址的服务器 IP。
检测结果如下图示。
![image](https://user-images.githubusercontent.com/4476837/154513531-11987b4d-5de9-4876-8a0e-ca6fa22faab1.png)

共计 106 个检测点，其中接受响应速度最快的节点在加拿大，目标服务器对其响应时间为 14ms；最慢在中国香港，响应时间 243ms。在成功访问的所有节点中，目标服务器的平均响应时间是 163.5ms。地图与颜色响应了国内不同省市的访问时间，红色说明访问超时，白色说明没有参与节点；约偏向绿色则响应时间越短。右侧还有一个统计表。这里的情况是：国内节点无法ping通github.com，即无法建立网络通讯。

**注意** 地图下方有一个独立 IP 的统计框，里面列举了所有节点被响应的 IP 地址。我们知道，网址到 IP 需要经过 DNS 解析，这里呈现的便是各个检测节点，经过默认的 DNS 解析得到的对应 IP 地址的汇总。事实上，你连接的网络也有默认或设定的 DNS，也会有一个解析得到的目标 IP，如果无法访问这个 IP 地址，就意味着你当前的网络无法访问github.com。

# 3. 使用本地ping工具,对IP地址进行ping测试
![image](https://user-images.githubusercontent.com/4476837/154514159-1a96d5ab-1f39-42b2-9e79-69c60439d151.png)
选择最合适的IP地址

# 4. 修改hosts,然后进行验证

再总结一下 Github 访问失败或者缓慢的原因：本机网络设置的 DNS 服务器解析 Github 相关域名到遭受污染的 IP 地址，这些 IP 地址要么本身无法访问，要么节点过远，从而导致了访问失败或者速度缓慢。

那么我们修改的方案可以是：

修改本机 Hosts 文件，主动建立域名与 IP 的映射关系，访问到这些域名时直接使用 Hosts 指定的 IP，绕过 DNS 解析。
修改网络的 DNS 服务器，换到能够解析出合适 IP 的 DNS 服务器。
显然第一种方案更加方便。因为 DNS 服务器储存的映射关系是动态更新的，无法直接控制。直接修改本机 Hosts 文件，锁定域名对应的 IP，更加有效方便。当然，Hosts 文件的作用就是绑定域名与 IP 的映射关系。

# 5. Windows 修改 Hosts 文件
打开 cmd，使用 vim 修改 Hosts 文件：
sudo vim C:\Windows\System32\drivers\etc\hosts
操作 vim 可以简单百度一下。添加 Github 相关域名的绑定，修改如上图所示，具体值见最后。
刷新网络 DNS 缓存：
ipconfig /flushdns
Windows 不自带sudo与vim，可以通过 Scoop 安装：scoop install sudo vim。

# 6 目前可用IP
列出当前使用的 Github 相关域名比较合适的 IP 值，笔者会定期维护更新。其中的设置可以解决github.com头像无法显示的问题：

```
# Github Hosts
# Update 20211204
# domain: github.com
140.82.114.4 github.com
140.82.114.10 nodeload.github.com
140.82.114.6 api.github.com
140.82.114.10 codeload.github.com
185.199.108.133 raw.github.com
185.199.108.153 training.github.com
185.199.108.153 assets-cdn.github.com
185.199.108.153 documentcloud.github.com
185.199.108.154 help.github.com

# domain: githubstatus.com
185.199.108.153 githubstatus.com

# domain: fastly.net
199.232.69.194 github.global.ssl.fastly.net

# domain: githubusercontent.com
185.199.108.133 raw.githubusercontent.com
185.199.108.154 pkg-containers.githubusercontent.com
185.199.108.133 cloud.githubusercontent.com
185.199.108.133 gist.githubusercontent.com
185.199.108.133 marketplace-screenshots.githubusercontent.com
185.199.108.133 repository-images.githubusercontent.com
185.199.108.133 user-images.githubusercontent.com
185.199.108.133 desktop.githubusercontent.com
185.199.108.133 avatars.githubusercontent.com
185.199.108.133 avatars0.githubusercontent.com
185.199.108.133 avatars1.githubusercontent.com
185.199.108.133 avatars2.githubusercontent.com
185.199.108.133 avatars3.githubusercontent.com
185.199.108.133 avatars4.githubusercontent.com
185.199.108.133 avatars5.githubusercontent.com
185.199.108.133 avatars6.githubusercontent.com
185.199.108.133 avatars7.githubusercontent.com
185.199.108.133 avatars8.githubusercontent.com
# End of the section
```
