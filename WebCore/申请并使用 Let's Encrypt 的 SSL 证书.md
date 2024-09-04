## 申请并使用 Let's Encrypt 的 SSL 证书

1. 启用 EPEL（Extra Packages for Enterprise Linux）仓库：
```bash
sudo dnf install epel-release
```
2. 安装 Certbot：
```bash
sudo dnf install certbot python3-certbot-nginx
```
3. 运行 Certbot 来获取或续期 SSL 证书：
```bash
sudo certbot --nginx -d www.3578.cn -d 3578.cn
```
## 设置自动续期
1. 添加域名解析 CAA 记录：
```base
主机记录：@
记录类型：CAA
  记录值：0 issue "letsencrypt.org"
```
1. 打开 crontab 来编辑定时任务：
```bash
sudo crontab -e
```
2. 添加 Certbot 续期任务。  
	在 crontab 文件中添加以下行，以每天两次检查并自动续期证书：
```bash
0 */12 * * * /usr/bin/certbot renew --quiet
```
	这行命令会在每天的 0 点和 12 点运行 certbot renew，并尝试续期任何快要过期的证书。
	--quiet 选项会减少输出，仅在发生错误时发送电子邮件通知。

3. 检查 Certbot 是否已经设置了自动续期任务。   
    对于使用 systemd 的系统，检查 Certbot 定时器的状态可以通过以下命令：
```bash
sudo systemctl list-timers | grep certbot
```
4. 使用 renew 命令来测试自动续期：
```bash
sudo certbot renew --dry-run
```
