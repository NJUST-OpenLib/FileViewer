# FileViewer

![GitHub License](https://img.shields.io/github/license/NJUST-OpenLib/FileViewer)


## 简介
**FileViewer** 是一款自用的纯前端的文件预览工具，无需后端支持，即可在浏览器中直接预览多种常见文件类型。  
该工具专为 [NJUST-docs](https://github.com/NJUST-OpenLib/NJUST-docs) 项目开发，可能并不适用于所有使用场景，且代码极其混乱，但仍然开放源码供参考。  
目前仍在开发中，已支持以下文件格式：

- 代码文件：支持 .css、.js、.m、.html、.c、.cpp 等所有 highlight.js 可解析的文本格式（不包括 .md）。
- 文档文件：`.pdf`、`.docx`
- Markdown 文件（`.md`）：可渲染为美观的 HTML 页面（其实一点也不美观，但最起码比Raw markdown 看起来顺眼多了）

## 功能特性
- **多格式支持**：支持代码文件、PDF、DOCX 和 Markdown 等格式
- **代码高亮**：使用 `highlight.js` 提供代码高亮，支持行号显示
- **纯前端实现**：无需后端服务，直接在浏览器中打开链接即可使用

## 使用方法
使用以下格式访问 FileViewer 进行文件预览：

```plaintext
http://example.com/autoviewer.html?url={文件链接}
```

将 `{文件链接}` 替换为实际文件的 URL，即可在浏览器中打开进行预览。

### 关于 PDF.js 的使用
这里引入 PDF.js 的方式可能不太优雅，直接把 `build` 和 `web` 目录全部复制过来了。同时可能缺少对音视频格式和其他 Office 格式的预览，但问题不大。

**跨域限制与调整**
由于 PDF.js 具有跨域限制，如果预览器和 PDF 文件的来源不同，可能会遇到加载失败的问题。这是由于 `/pdfjs/web/viewer.mjs` 中的 `validateFileURL` 代码所致：

```javascript
const HOSTED_VIEWER_ORIGINS = new Set(["null", "http://mozilla.github.io", "https://mozilla.github.io"]);
var validateFileURL = function (file) {
    if (!file) return;
    const viewerOrigin = URL.parse(window.location)?.origin || "null";
    if (HOSTED_VIEWER_ORIGINS.has(viewerOrigin)) return;
    const fileOrigin = URL.parse(file, window.location)?.origin;
    if (fileOrigin === viewerOrigin) return;

    throw new Error("file origin does not match viewer's");
};
```

为解决此问题，可直接修改该函数，使其跳过验证：

```javascript
var validateFileURL = function (file) {
    return; // 直接跳过验证，允许所有来源的 PDF 加载
};
```

## 部署步骤  

1. **下载代码**  
   ```bash
   git clone https://github.com/NJUST-OpenLib/FileViewer.git
   ```  

2. **上传文件**  
   将项目文件上传到 Web 服务器的指定目录。  

3. **注意跨域问题**  
   `autoviewer` 不支持直接通过本地文件访问，否则会遇到 CORS 问题。请使用 HTTP 服务器访问。

4. **访问预览器**  
   在浏览器中访问以下链接，即可预览文件：  
   ```plaintext
   http://example.com/autoviewer.html?url={文件链接}
   ```

## iframe 嵌入示例
这个工具主要是方便我直接 iframe 嵌入使用，以下是示例代码：

```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>URL 弹窗预览</title>
    <style>
        body {
            font-family: sans-serif;
            margin: 20px;
        }
        input[type="text"] {
            width: 300px;
            padding: 8px;
        }
        button {
            padding: 8px 12px;
            margin-left: 8px;
            cursor: pointer;
        }
        .modal {
            display: none;
            position: fixed;
            z-index: 1000;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.5);
        }
        .modal-content {
            background: #fff;
            margin: 5% auto;
            width: 80%;
            height: 80%;
            border: 1px solid #888;
            display: flex;
            flex-direction: column;
        }
        .modal-header {
            background-color: #f0f0f0;
            padding: 10px;
            border-bottom: 1px solid #888;
            position: relative;
        }
        .close {
            position: absolute;
            top: 50%;
            right: 15px;
            transform: translateY(-50%);
            font-size: 20px;
            font-weight: bold;
            cursor: pointer;
        }
        .modal-iframe-container {
            flex: 1;
        }
        .modal iframe {
            width: 100%;
            height: 100%;
            border: none;
        }
    </style>
</head>
<body>
    <input type="text" id="urlInput" placeholder="请输入 URL">
    <button id="openBtn">打开弹窗</button>
    <div id="modal" class="modal">
        <div class="modal-content">
            <div class="modal-header">
                <span class="close" id="closeBtn">&times;</span>
            </div>
            <div class="modal-iframe-container">
                <iframe id="modalIframe" src=""></iframe>
            </div>
        </div>
    </div>
    <script>
        const openBtn = document.getElementById('openBtn');
        const closeBtn = document.getElementById('closeBtn');
        const modal = document.getElementById('modal');
        const modalIframe = document.getElementById('modalIframe');
        const urlInput = document.getElementById('urlInput');

        openBtn.addEventListener('click', function () {
            const url = urlInput.value.trim();
            if (url) {
                modalIframe.src = 'http://example.com/autoview.html?url=' + encodeURIComponent(url);
                modal.style.display = 'block';
            } else {
                alert('请输入有效的 URL');
            }
        });

        closeBtn.addEventListener('click', function () {
            modal.style.display = 'none';
            modalIframe.src = '';
        });

        window.addEventListener('click', function (event) {
            if (event.target === modal) {
                modal.style.display = 'none';
                modalIframe.src = '';
            }
        });
    </script>
</body>
</html>
```


## 许可证
本项目基于 [MIT 许可证](LICENSE) 进行开源。


## 第三方库的使用

本项目使用以下库和资源来实现文件预览功能：  

- **[marked](https://github.com/markedjs/marked)**：用于将 Markdown 文件渲染成 HTML。

- **[mammoth](https://github.com/mwilliamson/mammoth.js)**：用于将 DOCX 文件转换为 HTML。

- **[highlight.js](https://github.com/highlightjs/highlight.js)**：用于代码高亮显示。
- 
- **[highlightjs-line-numbers.js](https://github.com/wcoder/highlightjs-line-numbers.js)**：用于为代码块添加行号。

- **[pdf.js](https://github.com/mozilla/pdf.js)**：用于预览 PDF 文件。

本项目使用 subframe7536 提供的字体 **[Maple Mono NF-CN](https://github.com/subframe7536/maple-font)** 
