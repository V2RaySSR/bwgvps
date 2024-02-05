## VPS库存监控系统
本程序主要用户推荐VPS，并监控一些商家的库存。

## 安装 Python 环境
```
apt update -y
apt upgrade -y
apt install python3 python3-pip git -y
```

## 安装/Install
```
# 拉取镜像
cd /root
git clone https://github.com/V2RaySSR/bwgvps.git && cd bwgvps
cp vpsmonitor/settings_local.py vpsmonitor/settings.py
pip3 install django && python3 manage.py makemigrations --merge && python3 manage.py migrate

# 创建后台超级用户
python3 manage.py createsuperuser

```
```
# 创建后台用户的回显
WARNINGS:
vps.Company: (models.W042) Auto-created primary key used when not defining a primary key type, by default 'django.db.models.AutoField'.
        HINT: Configure the DEFAULT_AUTO_FIELD setting or the VpsConfig.default_auto_field attribute to point to a subclass of AutoField, e.g. 'django.db.models.BigAutoField'.
vps.Goods: (models.W042) Auto-created primary key used when not defining a primary key type, by default 'django.db.models.AutoField'.
        HINT: Configure the DEFAULT_AUTO_FIELD setting or the VpsConfig.default_auto_field attribute to point to a subclass of AutoField, e.g. 'django.db.models.BigAutoField'.
vps.Passwd: (models.W042) Auto-created primary key used when not defining a primary key type, by default 'django.db.models.AutoField'.
        HINT: Configure the DEFAULT_AUTO_FIELD setting or the VpsConfig.default_auto_field attribute to point to a subclass of AutoField, e.g. 'django.db.models.BigAutoField'.
vps.Subscribe: (models.W042) Auto-created primary key used when not defining a primary key type, by default 'django.db.models.AutoField'.
        HINT: Configure the DEFAULT_AUTO_FIELD setting or the VpsConfig.default_auto_field attribute to point to a subclass of AutoField, e.g. 'django.db.models.BigAutoField'.
Username (leave blank to use 'root'): admin
Email address: admin@v2rayssr.com
Password: 
Password (again): 
Superuser created successfully.

```

## 开始启动服务
```
python3 manage.py runserver 0.0.0.0:8600
启动后，在浏览器中打开 http://127.0.0.1/admin 的“商品”及“商家”中填入数据，密码和用户为刚才创建的。
# /vps/models.py 为VPS配置数据（机房、线路等）

```

## 配置邮件
将 vpsmonitor/settings_local.py 重命名为 vpsmonitor/settings.py ，并设置好对应的邮箱信息。

## 定时监控/Crontab
定时每分钟检查VPS商家库存情况，需要在商家的属性中设定`检查库存`：`是`
然后将以下命令加入到crontab中。
```
crontab -e
# 末尾加入以下的两行
#check every 15 minutes for stock
*/15 * * * * /usr/bin/python3 /root/bwgvps/manage.py run >> /root/bwgvps/refreshing.log 2>&1 &
```


## 建立开机自启动
```
# systemd
cat > /etc/systemd/system/bwgvps.service <<EOF
[Unit]
Description=VPS_Stock_Monitor
After=network.target

[Service]
User=root
Type=simple
ExecStart=/usr/bin/python3 /root/bwgvps/manage.py runserver 0.0.0.0:8200
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl restart bwgvps.service
systemctl status bwgvps.service
systemctl enable bwgvps.service
```
