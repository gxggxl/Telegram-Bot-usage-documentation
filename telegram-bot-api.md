# 代理 telegram Bot API

在 Cloudflare 的 Workers 中部署（Deploy in Workers of cloudflare）

⚠️注意：仅能用于通知，互动还是用原生API吧。

```JavaScript
const whitelist = ["/bot"];
const tg_host = "api.telegram.org";
addEventListener('fetch', event => {
    event.respondWith(handleRequest(event.request))
})

function validate(path) {
    for (let i = 0; i < whitelist.length; i++) {
        if (path.startsWith(whitelist[i])) return true;
    }
    return false;
}

async function handleRequest(request) {
    let url = new URL(request.url);
    url.host = tg_host;
    if (!validate(url.pathname)) {
        return new Response('Unauthorized', {status: 403});
    }
    let req = new Request(`${url}`, {
        method: request.method,
        headers: request.headers,
        body: request.body
    });
    return await fetch(req);
}
```

你会得到类似`xxx.xxx.workers.dev`的URL。
