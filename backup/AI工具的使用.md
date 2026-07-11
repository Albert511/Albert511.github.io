# 一、codex
## 1、Windows中下载/安装
* 1、通过电脑中的微软商店搜索 ChatGPT安装；
* 2、通过官网下载安装；
   下载地址：https://get.microsoft.com/installer/download/9PLM9XGG6VKS?cid=website_cta_psi
## 2、使用
### 1、激活
#### 1.1、官方账号
如果有官方账号，并且订阅了对应的套餐，可直接打开codex使用官方账号登录；
#### 1.2、中转平台账号
1、第一步，如果打开了codex，先关掉codex进程，然后根据中转平台里的使用提示修改配置文件，如果还未打开过，就先打开codex然后再关闭（打开后codex才会初始化config.toml文件），找到并打开config.toml文件，将中转平台中的api秘钥里的配置复制到codex的配置文件前面；
`model_provider = "OpenAI"
model = "gpt-5.5"
review_model = "gpt-5.5"
model_reasoning_effort = "xhigh"
disable_response_storage = true
network_access = "enabled"
windows_wsl_setup_acknowledged = true

[model_providers.OpenAI]
name = "OpenAI"
base_url = "https://ailabhome.design2.org"
wire_api = "responses"
requires_openai_auth = true

[features]
goals = true`

<!-- Failed to upload "image.png" -->

2、第二步，选择通过其它方式登录codex，将OPENAI_API_KEY的值填入，登录即可；
`{
  "OPENAI_API_KEY": "sk-5b41060ac9b1c801b3fba2c44c3a12d6fb24471e6e3e703d4e1ab06f5174b1be"
}`
<img width="1637" height="1140" alt="Image" src="https://github.com/user-attachments/assets/9cbb1aa9-9bee-4a6c-b038-431deb7cdf5a" />

### 2、修改配置
主题，我更喜欢深色主题；
修改应用UI语言为中文；