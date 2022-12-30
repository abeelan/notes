[教程](https://readdevdocs.com/blog/makemoney/%E4%B8%AD%E5%9B%BD%E5%8C%BA%E6%B3%A8%E5%86%8COpenAI%E8%B4%A6%E5%8F%B7%E8%AF%95%E7%94%A8ChatGPT%E6%8C%87%E5%8D%97.html#%E5%89%8D%E6%9C%9F%E5%87%86%E5%A4%87)



进入页面：https://chat.openai.com

注册完成后出现提示：OpenAI’s services are not available in your country



在当前页面浏览器地址栏手动输入 `javascript: ` 然后粘贴下面内容，刷新页面即可。

```
window.localStorage.removeItem(Object.keys(window.localStorage).find(i=>i.startsWith('@@auth0spajs')))
```



失败了，需要购买国外手机号~~



平替网站：https://chatgpt.sbaliyun.com/

购买账号：https://2.ksfaka.com/chatgpt