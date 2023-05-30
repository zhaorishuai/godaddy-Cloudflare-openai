godaddy+Cloudflare代理openai补充
====
引用自：[使用 Cloudflare Workers 让 OpenAI API 绕过 GFW 且避免被封禁](https://github.com/noobnooc/noobnooc/discussions/9)，对其中内容进行补充，

首先，godaddy申请
-------
使用godaddy的好处在于：<br>  
1、域名相对便宜<br>  
2、支持支付宝支付，这个就很棒<br>  

godaddy申请域名的过程就不赘述了，之后就是按照[引用](https://github.com/noobnooc/noobnooc/discussions/9)中的步骤转移代理。

其次，Cloudflare代理
-------
输入godaddy申请的域名后，Cloudflare会自动检查，主要是第3步，将域名服务器配置到godaddy中。
![截图](https://github.com/zhaorishuai/godaddy-Cloudflare-openai/assets/39085989/48b59f76-03ef-4853-8403-009878743f43)

进入godaddy更改
![image](https://github.com/zhaorishuai/godaddy-Cloudflare-openai/assets/39085989/8bd12904-9d80-4434-9dd4-7d9a375ea17d)
更改之后，等待1个小时左右，域名就转移到Cloudflare下面了，然后就可以按照引用中的步骤继续进行了。
我使用的 Cloudflare Worker 的代码是：<br>  
const TELEGRAPH_URL = 'https://api.openai.com';

addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url);
  url.host = TELEGRAPH_URL.replace(/^https?:\/\//, '');

  const modifiedRequest = new Request(url.toString(), {
    headers: request.headers,
    method: request.method,
    body: request.body,
    redirect: 'follow'
  });

  const response = await fetch(modifiedRequest);
  const modifiedResponse = new Response(response.body, response);

  // 添加允许跨域访问的响应头
  modifiedResponse.headers.set('Access-Control-Allow-Origin', '*');

  return modifiedResponse;
}

最后，Cloudflare的坑
-------
我的代理不知道为啥突然不好使了，网上各种查也没查到原因，还以为被封了，后来我才看明白：Cloudflare的域名服务器是会变的！！！<br>
所以当域名突然失效了，看看Cloudflare的域名服务器跟godaddy中配置的是否一致，不一致的话再更改下godaddy的域名服务器就可以了
