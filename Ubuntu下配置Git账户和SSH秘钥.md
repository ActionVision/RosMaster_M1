# Ubuntu 下配置 Git 账户和 SSH 秘钥

---

## 一、配置 Git 用户名和邮箱

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱@example.com"

# 查看配置
git config --global --list
```

---

## 二、生成 SSH 秘钥

```bash
# 生成秘钥（一路回车即可）
ssh-keygen -t ed25519 -C "你的邮箱@example.com"

# 如果系统不支持 ed25519，用 RSA：
# ssh-keygen -t rsa -b 4096 -C "你的邮箱@example.com"
```

默认保存在 `~/.ssh/id_ed25519`（私钥）和 `~/.ssh/id_ed25519.pub`（公钥）。

---

## 三、添加公钥到 GitHub / Gitee

```bash
# 查看公钥内容
cat ~/.ssh/id_ed25519.pub
```

复制输出的全部内容，然后：

- **GitHub**: Settings → SSH and GPG keys → New SSH key → 粘贴
- **Gitee**: 设置 → SSH 公钥 → 粘贴

---

## 四、测试连接

```bash
# GitHub
ssh -T git@github.com

# Gitee
ssh -T git@gitee.com
```

看到 `Hi xxx! You've successfully authenticated` 就成功了。

---

## 五、多个平台共存（GitHub + Gitee）

```bash
# 编辑 SSH 配置
vim ~/.ssh/config
```

写入：

```
# GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github

# Gitee
Host gitee.com
    HostName gitee.com
    User git
    IdentityFile ~/.ssh/id_ed25519_gitee
```

然后分别为两个平台生成独立的秘钥。

---

## 六、启动 ssh-agent 自动加载秘钥

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519