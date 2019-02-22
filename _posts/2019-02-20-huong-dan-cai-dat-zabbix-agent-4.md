---
date: 2019-02-20
title: Hướng dẫn cài đặt zabbix agent 4.0
categories:
  - Monitor
description: Hướng dẫn cài đặt zabbix agent lên host cần monitor chạy trên các hệ điều hành Centos, Ubuntu, Linux.
author: duydm
tags: [Zabbix]
type: Document
---

Sau khi triển khai <a href="https://blog.cloud365.vn/monitor/cai-dat-zabbix-4-lts-tren-centos7/" target="_blank">zabbix server</a> để có thể giám sát được các host client khác phải cài đặt `zabbix-agent` lên client, tùy theo cơ chế lựa chọn (active check, passive check, SSH check...) cho việc nhận metric từ các host. Ở bài hướng dẫn này giới thiệu cho bạn các bước để cài đặt `zabbix-agent 4.0` lên host client trên hệ điều hành khác nhau (Windows, <a href="https://blog.cloud365.vn/linux/huong-dan-cai-dat-centos7/" target="_blank">CentOS server</a>, Ubuntu) và giám sát thông số trên zabbix server.

### Mục lục

[1. Chuẩn bị](#chuanbi)<br>
[2. Cài đặt zabbix agent](#setup)<br>
[3. Cấu hình zabbix agent](#config)<br>
[4. Add host zabbix agent lên zabbix server](#host)<br>

<a name="chuanbi"></a>
## 1. Chuẩn bị

**Mô hình triển khai**

![](/images/img-agent-zabbix/topo-agent.png)

**IP planning**

![](/images/img-agent-zabbix/Screenshot_978.png)

<a name="setup"></a>
## 2. Cài đặt zabbix agent

### 2.1. Cài đặt zabbix agent trên Linux

**2.1.1. Host Centos**

**Truy cập host CentOS `10.10.10.119`**

![](/images/img-agent-zabbix/Screenshot_1001.png)

**Thêm zabbix agent repository**

CentOS 7 / RHEL 7

```
rpm -ivh https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm
```

CentOS 6 / RHEL 6

```
rpm -ivh https://repo.zabbix.com/zabbix/4.0/rhel/6/x86_64/zabbix-release-4.0-1.el6.noarch.rpm
```

**Cài đặt**

```
yum install -y zabbix-agent
```

**Check zabbix-agent**

```
zabbix_agentd -V
```
![](/images/img-agent-zabbix/Screenshot_981.png)

**2.1.2. Host Ubuntu**

**Truy cập host CentOS `10.10.10.105`**

![](/images/img-agent-zabbix/Screenshot_983.png)

**Thêm zabbix agent repository**

Ubuntu 18.04

```
wget https://repo.zabbix.com/zabbix/4.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_4.0-2+bionic_all.deb
dpkg -i zabbix-release_4.0-2+bionic_all.deb
apt-get update
```

Ubuntu 16.04

```
wget https://repo.zabbix.com/zabbix/4.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_4.0-2+xenial_all.deb
dpkg -i zabbix-release_4.0-2+xenial_all.deb
apt-get update
```

**Cài đặt**

```
apt-get install -y zabbix-agent
```

**Check zabbix-agent**

```
zabbix_agentd -V
```
![](/images/img-agent-zabbix/Screenshot_984.png)

### 2.1. Cài đặt zabbix agent trên Windows

+ Cài đặt

**Truy cập host Windows `10.10.10.109`**

![](/images/img-agent-zabbix/Screenshot_990.png)

**Download zabbix agent Windows**

Truy cập đường link <a href="http://www.suiviperf.com/zabbix/old/" target="_blank">tại đây</a>.

Lưu ý lựa chọn phiên bản phù hợp với hệ điều hành để tải về.

![](/images/img-agent-zabbix/Screenshot_988.png)


**Cài đặt**

![](/images/img-agent-zabbix/Screenshot_991.png)
![](/images/img-agent-zabbix/Screenshot_992.png)

Đồng ý với điều khoản nhà cung cấp

![](/images/img-agent-zabbix/Screenshot_993.png)

Nhập thông tin zabbix server muốn kết nối tới.

![](/images/img-agent-zabbix/Screenshot_994.png)

Quá trình cài đặt

![](/images/img-agent-zabbix/Screenshot_995.png)
![](/images/img-agent-zabbix/Screenshot_996.png)
![](/images/img-agent-zabbix/Screenshot_997.png)

Kết thúc cài đặt

![](/images/img-agent-zabbix/Screenshot_998.png)

+ Kiểm tra service zabbix-agent

Mở cửa sổ `cmd` -> Nhập `services.msc`

![](/images/img-agent-zabbix/Screenshot_999.png)

<a name="config"></a>
## 3. Cấu hình zabbix agent

Để zabbix client có thể gửi metric về zabbix server ta phải thay đổi cấu hình trong file config của zabbix agent để client biết gửi về server nào.

### 3.1. Đối với host Linux

Trên host chạy hệ điều hành Linux file cấu hình zabbix agent được đặt ở `/etc/zabbix/zabbix_agentd.conf`. Thực hiện các câu lệnh dưới để mở port `10050`, chỉ định `IP zabbix server`.<br>
**Lưu ý:** Sau mỗi thay đổi config zabbix agent bạn phải khởi đông lại service `zabbix-agent`.

```
cp /etc/zabbix/zabbix_agentd.conf /etc/zabbix/zabbix_agentd.conf.bk
sed -i 's/Server=127.0.0.1/Server=10.10.10.115/g' /etc/zabbix/zabbix_agentd.conf
sed -i 's/# ListenPort=10050/ListenPort=10050/g' /etc/zabbix/zabbix_agentd.conf
sed -i 's/ServerActive=127.0.0.1/ServerActive=10.10.10.115/g' /etc/zabbix/zabbix_agentd.conf
systemctl enable zabbix-agent
systemctl start zabbix-agent
systemctl restart zabbix-agent
systemctl status zabbix-agent
```

![](/images/img-agent-zabbix/Screenshot_985.png)

### 3.1. Đối với host Windows

Khi bạn muốn thay đổi cấu hình `zabbix-agent` thay đổi file `zabbix_agentd.conf` ở thư mục `C:\Program Files\Zabbix Agent`

![](/images/img-agent-zabbix/Screenshot_1000.png)

**Lưu ý:** Sau mỗi thay đổi config zabbix agent bạn phải khởi đông lại service `zabbix-agent`.

<a name="host"></a>
## 4. Add host zabbix agent lên zabbix server

### 4.1. Truy cập zabbix server

![](/images/img-agent-zabbix/Screenshot_1002.png)

### 4.2. Add host

Trên web dashboard của zabbix server click `Configuration -> Hosts -> Create Host -> Host`

![](/images/img-agent-zabbix/Screenshot_1003.png)

**Nhập thông tin host client**

```
Host name:
Group:
Agent interfaces:
```
![](/images/img-agent-zabbix/Screenshot_1006.png)

**Lựa chọn template**

Chuyển sang tab `Temaplates` -> `Select`

![](/images/img-agent-zabbix/Screenshot_1007.png)

Lựa chọn `Template` phù hợp với client của bạn.

Click `Add`

![](/images/img-agent-zabbix/Screenshot_1008.png)

**Add host thành công**

![](/images/img-agent-zabbix/Screenshot_1009.png)

Chờ một lúc để client kết nối tới zabbix server. Khi biểu tượng zabbix agent đổi màu xanh là ta đã add host thành công.

![](/images/img-agent-zabbix/Screenshot_1010.png)

**Kiểm tra thông số monitor**

Click `Monitoring -> Lastest data -> Lựa chọn host -> Apply`

![](/images/img-agent-zabbix/Screenshot_1011.png)

Hy vọng những hướng dẫn trên giúp bạn có thể cài đặt thành công zabbix-agent và giám sát được các thông số của client.

---
Thực hiện bởi <a href="https://cloud365.vn/" target="_blank">cloud365.vn</a>