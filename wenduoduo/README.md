# Wenduoduo SDK

文多多SDK集成说明

## 使用方法

#iFrame 方案
文多多 AiPPT 可以通过 iframe 与您的系统紧密结合，通过简单的一些步骤，就能将 Docmee 嵌入到您的业务系统中。
快速使用您的 TOKEN 来体验 iframe 嵌入的效果

#准备
下载我们提供的 SDK 文件 
在您的页面中引入
    <head>
    <script src="docmee-ui-sdk-iframe.min.js"></script>
    </head>

#初始化
您需要实例化我们提供的类 DocmeeUI 来嵌入我们的 UI
⚠️ 不要在 file 协议下运行，请启动一个 http 服务来运行！
接入代码示例
github： https://github.com/veasion/aippt-ui-iframe
gitee： https://gitee.com/veasion/aippt-ui-iframe
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <title>文多多 AiPPT</title>
    <script src="docmee-ui-sdk-iframe.min.js"></script>
    <style>
      body {
        width: 100vw;
        height: 100vh;
        margin: 0;
        display: flex;
        justify-content: center;
        align-items: center;
      }
      #container {
        width: calc(100% - 20px);
        height: calc(100% - 20px);
        margin: 0;
        padding: 0;
        border-radius: 12px;
        box-shadow: 0 0 12px rgba(120, 120, 120, 0.3);
        overflow: hidden;
        background: linear-gradient(-157deg, #f57bb0, #867dea);
        color: white;
      }
    </style>
  </head>
  <body>
    <div id="container"></div>
  </body>
  <script>
    // 请在服务端调用 createApiToken 接口生成token（不同uid生成的token数据相互隔离）
    // 接口文档：https://docmee.cn/open-platform/api#%E5%88%9B%E5%BB%BA%E6%8E%A5%E5%8F%A3-token
    var token = createApiToken();

    // 初始化 UI iframe
    const docmeeUI = new DocmeeUI({
      pptId: null,
      token: token, // token
      container: document.querySelector("#container"), // 挂载 iframe 的容器
      page: "creator", // 'creator' 创建页面; 'dashboard' PPT列表; 'customTemplate' 自定义模版; 'editor' 编辑页（需要传pptId字段）
      lang: "zh", // 国际化
      mode: "light", // light 亮色模式, dark 暗色模式
      isMobile: false, // 移动端模式
      background: "linear-gradient(-157deg,#f57bb0, #867dea)", // 自定义背景
      padding: "40px 20px 0px",
      onMessage(message) {
        console.log("监听事件", message);
        if (message.type === "invalid-token") {
          // 在token失效时触发
          console.log("token 认证错误");
          // 更换新的 token
          // let newToken = createApiToken()
          // docmeeUI.updateToken(newToken)
        } else if (message.type === "beforeGenerate") {
          const { subtype, fields } = message.data;
          if (subtype === "outline") {
            // 生成大纲前触发
            console.log("即将生成ppt大纲", fields);
            return true;
          } else if (subtype === "ppt") {
            // 生成PPT前触发
            console.log("即将生成ppt", fields);
            docmeeUI.sendMessage({
              type: "success",
              content: "继续生成PPT",
            });
            return true;
          }
        } else if (message.type === "beforeCreateCustomTemplate") {
          const { file, totalPptCount } = message.data;
          // 是否允许用户继续制作PPT
          console.log("用户自定义完整模版，PPT文件：", file.name);
          if (totalPptCount < 2) {
            console.log("用户积分不足，不允许制作自定义完整模版");
            return false;
          }
          return true;
        } else if (message.type == "pageChange") {
          pageChange(message.data.page);
        } else if (message.type === "beforeDownload") {
          // 自定义下载PPT的文件名称
          const { id, subject } = message.data;
          return `PPT_${subject}.pptx`;
        } else if (message.type == "error") {
          if (message.data.code == 88) {
            // 创建token传了limit参数可以限制使用次数
            alert("您的次数已用完");
          } else {
            alert("发生错误：" + message.data.message);
          }
        }
      },
    });
  </script>
</html>

#参数说明
参数名称	类型	必填	说明	例
token	string	✔︎	调用 API 创建接口 token 获取 token	sk_xxx
container	HTMLElement	✔︎	挂载 iframe 的容器	
themeColor	string	⨯	主题色	#4b39b8
page	'dashboard' | 'creator' | 'editor' | 'customTemplate'	⨯	进入页面，'dashboard'表示文档列表页，'creator'生成 ppt 页面, 'customTemplate'表示进入自定义模版页面，'editor'表示编辑页面（pptId 必须同时传递）	dashboard
lang	'zh' | 'en' | 'jp' | 'de' | 'fr' | 'ko' | 'pt'	⨯	语言(详见 国际化)	'zh'
pptId	string	○	进入 editor 页面时编辑的 pptId，如果 page 为 editor 时，pptId 不能为空	-
background	string	⨯	iframe 背景颜色，可填入颜色或者图片 url 地址	#f1f1f1
mode	'light' , 'dark'	⨯	亮色，暗色模式	当前浏览器环境
isMobile	boolean	⨯	移动端模式	false
backgroundSize	string	⨯	iframe 背景大小 与 CSS 中的 background-size 语法相同	cover
padding	string	⨯	内边距（也就是 css 的 padding，语法相同）	20px 10px 20px 10px
onMessage	function	⨯	事件处理钩子 详见 事件类型	function (message) {}
creatorData	{subject: string, createNow?: boolean} | {text: string, createNow?: boolean}	⨯	生成页面传递 内容 subject 与 text 二选其一； createNow 如果为 true表示直接开始大纲生成（仅当 page=creator）时生效	-
downloadButton	boolean | ['pptx', 'pdf']	⨯	下载文件选项 返回 false 表示禁用下载，如果只想打开一种下载方式，可以传递数组['pptx']表示只允许下载为 pptx 格式	true
creatorMode	['topic', 'material']	⨯	生成 PPT 方式，topic：主题生成，material：外部资料	['topic', 'material']
outlineExportFormat	'txt'| 'md'	⨯	导出大纲的文件格式（注意：不管是 txt 还是 md 格式，内容都是按照 markdown 语法来导出的）	'md'
createCustomTemplateWhenEmpty	boolean	⨯	控制自定义模版选择界面，若自为空时，是否需要显示“立即创建”按钮	false
createCustomTemplateWhenSelect	boolean	⨯	控制自定义模版选择界面，是否显示“自定义模版”按钮，若设置为 true，则 createCustomTemplateWhenEmpty 参数无效	false
css	string	⨯	注入 CSS 样式，可可通过传递自定义 CSS 来更加深度地自定义文多多的样式，支持可访问的 URL 地址或直接传入 CSS 字符串，例如：#docmee_SdkContainer {background: white !important;}，或'https://abc.cn/style.css'	-

#事件类型
iFrame 挂载完成
{
  type: 'mounted',
}
PPT 生成前触发 (用户点击“生成大纲”，以及选择完模版，点击“开始创作”时触发,通 过 subtype 区分) 需返回布尔值
{
  type: 'beforeGenerate',
  data: {
    subtype: 'outline', // "outline" 生成大纲前触发，"ppt"生成ppt前触发
    fields: {} // 用户用于生成PPT的参数
  }
}
该事件可以返回 true/false (也可以返回 异步 Promise<boolean>) 来决定用户是否能 够继续生成 注意
自定义完整模版前触发 需返回布尔值
{
  type: 'beforeCreateCustomTemplate',
  data: {
    file: {}, // 用户上传的PPT文件
    totalPptCount: 99 // 用户剩余的积分数
  }
}
自定义模版完成触发
{
  type: 'afterCreateCustomTemplate',
  data: {
    do: 'create_complex', // 创建完整自定义模版 (积分 -2)
    do: 'modify_complex', // 修改完整自定义模版  (不扣积分)
    do: 'create_simple', // 创建简单自定义模版 (不扣积分)
    ...
    // 自定义模版数据
  }
}
PPT 生成完毕扣费时触发
{
  type: 'charge',
  data: {
    id: 'xxxxxx' // ppt id
  }
}
PPT 生成后触发
{
  type: 'afterGenerate',
  data: {
    id: 'xxxxxx' // ppt id
  }
}
下载前触发 需返回布尔值或字符串（文件名）
{
  type: 'beforeDownload',
  data: {
    id: 'xxxxxx' // ppt id
  }
}
该事件可以返回 true/false 或 string (也可以返回 异步 Promise<boolean | string>) 来决定用户是否能够继续下载 PPT，或指定 ppt 文档名称，返回的名称需要 以.pptx结尾才表示重命名文件 注意
用户信息
{
  type: 'user-info',
  data: {
   "uid": null, // 用户 id
   "availableCount": 0, // 可用生成次数
   "usedCount": 0 // 已使用的次数
  }
}
错误
{
  type: 'error',
  data: {
    code: 88, // 次数用完了
    message:"您的次数已用完，请开通会员"
  }
}

#其他 API
以下方法都是 DocmeeUI 类 实例的成员，需要通过docmeeUI.来调用
docmeeUI.updateToken(newToken: string): void 更新用户 Token
docmeeUI.destroy(): void 卸载 iframe
docmeeUI.getInfo(): void 手动获取一次 用户信息，用户信息会在onMessage回调 中返回
docmeeUI.navigate(obj: {page: 'creator' | 'dashboard' | 'editor' | 'customTemplate', pptId?: string}): void 跳转页面, 同样地如果前往 editor 页面，pptId 是必须的
docmeeUI.navigate({ page: "dashboard" });
docmeeUI.navigate({ page: "editor", pptId: "xxxx" });
docmeeUI.sendMessage(message: {type: string, content: string}): void 控制 SDK 发送消息
/**
 *  type: 'success' | 'error' | 'warning' | 'info' | undefined
 *  content: string
 */
docmeeUI.sendMessage({ type: "success", content: "操作成功" });
docmeeUI.changeCreatorData(data: {subject: string, text: string}, createNow: boolean): void 在creator页面中修改输入框中的值
/**
 *  参数1: 用来生成大纲的数据 subject(主题) 或 text(文字内容) 二选其一
 *  参数2: 控制是否立即生成
 *    如果为true，表示方法调用时直接开始生成，如果不传递或者传递false时，仅输入内容，需要用户点击生成大纲按钮
 */
docmeeUI.changeCreatorData({ subject: "AI未来的发展" }, true);
// 或
docmeeUI.changeCreatorData({ text: "AI未来的发展" }, true);
docmeeUI.updateTemplate(templateId: string) 外部指定更换模板，并刷新
docmeeUI.showTemplateDialog(type?: 'custom' | 'system') 弹出模板选择弹框, type: 'custom' or 'system' (default)
docmeeUI.getCurrentPptInfo() 返回 ppt 信息（在事件中返回，事件类型currentPptInfo）

#国际化
为了应对多语种环境，文多多 AiPPT 支持国际化。
目前支持的语言列表有:
中文 zh
英文 en
日本语 jp
韩语 ko
法语 fr
德语 de
葡萄牙语 pt

#接口转发
如果你想对某些接口进行特殊处理，比如图片接口走你们自己的图库接口之类的扩展，或者 API 代理商 提供给用户 iframe 接入方式，都可以通过 nginx 进行接口转发实现。
示例：
假设你的服务器域名为 xxx.com
服务器端 nginx 配置如下：
server {
    listen 8080;
    server_name xxx.com;
    charset utf-8;
    # 图片接口特殊处理，走你们自己的图片接口逻辑
    # location /api/ppt/genImg {
    #     proxy_pass http://127.0.0.1/api/ppt/genImg;
    #     proxy_set_header Host $host;
    #     proxy_set_header X-Real-IP $remote_addr;
    #     proxy_set_header X-Forwarded-Proto $scheme;
    #     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #     proxy_set_header X-Real-Port $remote_port;
    # }
    # 接口代理
    location ^~ /api {
        proxy_pass https://docmee.cn/api;
        proxy_read_timeout 600;
        proxy_connect_timeout 600;
        proxy_send_timeout 600;
        proxy_cache off;
        proxy_buffering off;
        proxy_http_version 1.1;
        proxy_set_header Host docmee.cn;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto 'https';
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-Port $remote_port;
    }
    # 页面代理
    location / {
        proxy_pass https://docmee.cn;
        proxy_set_header Host docmee.cn;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto 'https';
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-Port $remote_port;
    }
}
使用 DocmeeUI 是指定代理相关参数：
const docmeeUI = new DocmeeUI({
    // 前端域名代理
    DOMAIN: 'http://xxx.com:8080',
    // 服务端代理
    baseURL: 'http://xxx.com:8080/api',
    ....
})