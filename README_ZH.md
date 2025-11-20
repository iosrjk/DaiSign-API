[English说明](README.md) | [中文说明](README_ZH.md)

# IPA 签名 API (所有代码都是使用 ChatGPT O1 Mini/O1 (还有 O3 mini >~<) 编写的，向你们致敬 :3)

## IPA 文件最大上传大小为 2GB，如果你想在自己的网站/本地使用，可以在代码中修改这个限制

## 如何安装

1. **克隆仓库**
   ```bash
   git clone https://github.com/daisuke1227/DaiSign-API.git
   cd DaiSign-API
   ```
1.5 **仅限 Linux/Mac 用户，使 "zsign" 文件可执行**   
```bash
chmod +x zsign
```

2. **安装 Node.js 和 npm**
   ```bash
   sudo apt update
   sudo apt install nodejs npm -y
   ```

3. **（可选但推荐）更新 Node.js 到最新版本**
   ```bash
   sudo npm install -g n
   sudo n latest
   ```

4. **安装项目依赖**
   ```bash
   npm install
   ```

5. **配置环境变量**

   将 .env.example 重命名为 .env 并填写配置信息：
   ```bash
   mv .env.example .env
   ```

6. **启动服务器**
   ```bash
   node app.js
   ```
欢迎使用签名 API！这个 API 允许你上传 IPA 文件、.p12 证书和配置文件，然后返回已签名应用的安装链接。

## 功能特性
- 上传IPA文件进行签名。
- 使用`.p12`证书和配置文件来签名应用。
- 获取已签名IPA的安装链接。

## 工作原理
1. 向 `/sign` 发送 POST 请求并上传必需的文件。
2. 服务器处理文件并对 IPA 进行签名。
3. 收到安装链接以下载和安装已签名的应用。

### 在本地使用
如果你想在自己的网站使用这个功能，请修改 app.js 中的这一行： ```const UPLOAD_URL = 'https://yoursite.com/'; // 更新为你的实际域名``` 然后按照教程使用你自己的网站（这部分是 Daisuke 在发布后编写的） 

### 修改默认 IPA
在 app.js 代码中，如果你没有上传 IPA，将会自动使用一个默认的 IPA。你可以在 ```const DEFAULT_IPA_PATH = path.join(__dirname, 'Portal-1.9.0.ipa'); // 确保此文件存在``` 中修改这个设置。代码会检查 IPA 文件是否存在，如果用户没有上传 IPA，就会使用这个默认 IPA。你可以在自己的代码中设置 IPA 为必需项来绕过这个功能。

## API 端点

### POST `/sign`

#### 参数

| 名字            | 类型   | 是否必需 | 描述                                    |
|------------------|--------|----------|--------------------------------------------------|
| `ipa`           | 文件   | 否       | 要签名的 IPA 文件。如果使用默认 IPA 则为可选。 |
| `p12`           | 文件   | 是      | 用于签名的`.p12` 证书文件。         |
| `mobileprovision` | 文件   | 是      | 配置文件文件。                 |
| `p12_password`  | 字符串 | 否       |  `.p12` 证书的密码（如果需要）。|

## 使用示例

### Curl
```bash
curl -X POST https://api.daisign.lol/sign \
-F "ipa=@/path/to/app.ipa" \
-F "p12=@/path/to/cert.p12" \
-F "mobileprovision=@/path/to/profile.mobileprovision" \
-F "p12_password=your_password"
```
### Python
```python
import requests

url = "https://api.daisign.lol/sign"
files = {
    'ipa': open('/path/to/app.ipa', 'rb'),
    'p12': open('/path/to/cert.p12', 'rb'),
    'mobileprovision': open('/path/to/profile.mobileprovision', 'rb'),
}
data = {'p12_password': 'your_password'}

response = requests.post(url, files=files, data=data)
print(response.json())
```
### JavaScript (Node.js) 
```javascript
const FormData = require('form-data');
const axios = require('axios');
const fs = require('fs');

const form = new FormData();
form.append('ipa', fs.createReadStream('/path/to/app.ipa'));
form.append('p12', fs.createReadStream('/path/to/cert.p12'));
form.append('mobileprovision', fs.createReadStream('/path/to/profile.mobileprovision'));
form.append('p12_password', 'your_password');

axios.post('https://api.daisign.lol/sign', form, {
    headers: form.getHeaders(),
})
.then(response => console.log(response.data))
.catch(error => console.error(error));
```

### HTML/JS (用于网站和基础模板)
```
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>IPA 签名工具</title>
</head>
<body>
    <h2>IPA 签名工具</h2>
    <form id="signForm" action="https://api.daisign.lol/sign" method="POST" enctype="multipart/form-data">
        <p>
            <label for="ipa">IPA 文件 (.ipa) <small>（可选）</small></label><br>
            <input type="file" id="ipa" name="ipa" accept=".ipa">
        </p>
        <p>
            <label for="p12">证书文件 (.p12) <small>（必需）</small></label><br>
            <input type="file" id="p12" name="p12" accept=".p12" required>
        </p>
        <p>
            <label for="mobileprovision">配置文件 (.mobileprovision) <small>（必需）</small></label><br>
            <input type="file" id="mobileprovision" name="mobileprovision" accept=".mobileprovision" required>
        </p>
        <p>
            <label for="p12_password">P12 密码 <small>（可选）</small></label><br>
            <input type="password" id="p12_password" name="p12_password" placeholder="输入 P12 密码">
        </p>
        <p>
            <button type="submit">签名 IPA</button>
        </p>
    </form>

    <!-- 弹窗容器 -->
    <div id="popupContainer"></div>

    <script>
        document.addEventListener("DOMContentLoaded", () => {
            const form = document.getElementById("signForm");

            const popupContainer = document.getElementById("popupContainer");

            const adjustPopupSize = () => {
                popupContainer.style.width = window.innerWidth < 600 ? "80%" : "400px";
                popupContainer.style.maxWidth = "90%";
                popupContainer.style.top = "50%";
                popupContainer.style.left = "50%";
                popupContainer.style.transform = "translate(-50%, -50%)";
                popupContainer.style.position = "fixed";
                popupContainer.style.backgroundColor = "#fff";
                popupContainer.style.border = "1px solid #ccc";
                popupContainer.style.boxShadow = "0 2px 10px rgba(0, 0, 0, 0.2)";
                popupContainer.style.zIndex = "1000";
                popupContainer.style.padding = "10px";
                popupContainer.style.textAlign = "center";
            };

            window.addEventListener("resize", adjustPopupSize);

            form.addEventListener("submit", async (e) => {
                e.preventDefault();
                const formData = new FormData(form);

                try {
                    const response = await fetch(form.action, {
                        method: "POST",
                        body: formData,
                    });

                    if (!response.ok) {
                        throw new Error("IPA 签名失败，请重试。");
                    }

                    const data = await response.json();

                    if (data.installLink) {
                        popupContainer.innerHTML = `
                            <h3>签名完成</h3>
                            <p>您的已签名 IPA 已准备就绪。点击下方链接安装：</p>
                            <a href="${data.installLink}" target="_blank">${data.installLink}</a>
                            <br><br>
                            <button onclick="document.getElementById('popupContainer').style.display='none'">关闭</button>
                        `;
                        popupContainer.style.display = "block";
                        adjustPopupSize();
                    } else {
                        throw new Error("服务器返回了无效的响应。");
                    }
                } catch (error) {
                    popupContainer.innerHTML = `
                        <h3>错误</h3>
                        <p>${error.message}</p>
                        <button onclick="document.getElementById('popupContainer').style.display='none'">关闭</button>
                    `;
                    popupContainer.style.display = "block";
                    adjustPopupSize();
                }
            });
        });
    </script>
</body>
</html>
```
### 响应示例
```
{
  "installLink": "itms-services://?action=download-manifest&url=https://api.daisign.lol/plist/example.plist"
}
```
|字段	|	描述|
|------------------|--------|
|installLink	| 下载并安装已签名应用程序的链接。|

常见错误

缺少必需文件

{
  "error": "P12 and MobileProvision files are required."
}

	•	原因： 没有上传 .p12 或 .mobileprovision 文件。
	•	解决方法： 确保请求中包含这两个文件。


注意事项

	•	IPA 文件是可选的。如果没有提供，将使用默认的 IPA。
	•	p12_password 仅在你的证书有密码时才需要。


致谢

	•	API 开发者： Daisuke
	•	文档协助： ChatGPT
	•	文档翻译： iosrjk
  

有问题或遇到困难？

如果你有任何问题或遇到困难，可以通过 [Discord](https://discord.com/users/630151942135480370) 或 [Telegram](https://t.me/dai1228)联系我。
