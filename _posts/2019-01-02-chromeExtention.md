---
layout: post
title: 自定义chrome扩展程序
category: utility
tags: [chrome extention]
---

# 开发chrome扩展程序

当我们访问web网站，发现页面的样式不符合我们的需求，我们可以通过chorme扩展程序修改页面。

首先我们需要准备一个清单文件-manifest.json，告诉chrome浏览器你的扩展程序的基本信息。创建一个目录包含manifest.json文件，文件内容如下：

```text
{
  "manifest_version": 2,
  "name": "test",
  "version": "0.1"
}
```

### 加载扩展程序的到Chrome

打开`chrome://extensions/` ，开启开发者模式，加载已解压的扩展程序，选择包含manifest.json文件的目录。

### content scripts

内容脚本是js文件，可以与访问的页面进行交互。让我们加入一个简单地js:

```text
// content.js
alert("Hello from your Chrome extension!")
```

我们需要告诉manifest.json javascript文件的信息。

```text
"content_scripts": [
  {
    "matches": [
      "<all_urls>"
    ],
    "js": ["content.js"]
  }
]
```



`<all_urls>`代表每访问一个页面就加载js文件。如果需要制定页面加载可以替换成相应的地址。\["[https://mail.google.com/\*](https://mail.google.com/*)", "[http://mail.google.com/\*](http://mail.google.com/*)"\] 可以通过类似正则表达式匹配多个页面。

现在我们可以加载这个扩展程序，每次访问页面都会提示"`Hello from your Chrome extension!"`

## 进阶

### [Browser Actions](https://thoughtbot.com/blog/how-to-make-a-chrome-extension#browser-actions) <a id="browser-actions"></a>

#### 添加一个图标给扩展程序

```text
"browser_action": {
  "default_icon": "icon.png"
}
```

### [Message passing](https://thoughtbot.com/blog/how-to-make-a-chrome-extension#message-passing) <a id="message-passing"></a>

扩展程序可以监听鼠标事件，但是content script只能操作访问的页面，不能监听扩展程序的点击事件。因此我们需要添加新的脚本。

```text
"background": {
  "scripts": ["background.js"]
}
```

```text
// background.js

// Called when the user clicks on the browser action.
chrome.browserAction.onClicked.addListener(function(tab) {
  // Send a message to the active tab
  chrome.tabs.query({active: true, currentWindow: true}, function(tabs) {
    var activeTab = tabs[0];
    chrome.tabs.sendMessage(activeTab.id, {"message": "clicked_browser_action"});
  });
});
```


* * *

background.js脚本可以访问chrome api,但不能访问当前页面，它会和content script进行交互，完成监听鼠标事件到处理当前页面的过程。现在修改一下content.js. 

```text
// content.js
chrome.runtime.onMessage.addListener(
  function(request, sender, sendResponse) {
    if( request.message === "clicked_browser_action" ) {
      
      
 var el = document.getElementsByName('isExecute');
	   
	var len = el.length;
	
	for(var i=0; i<len; i++)
	{
		    console.log("start to release")
			el[i].checked = false;
		
	}
      
    }
  }
);
```


完整的结构如下：
![](/images/posts/utility/structure.png)


manifest完整内容：

```text
{
  "manifest_version": 2,
  "name": "lanjing job extention",
  "version": "0.1",
  "background": {
    "scripts": ["background.js"]
  },
   "browser_action": {
    "default_icon": "icon.png"
  },
  "content_scripts": [
  {
    "matches": [
      "<all_urls>"
    ],
    "js": ["jquery-min.js","content.js"]
  }
]
}
```

