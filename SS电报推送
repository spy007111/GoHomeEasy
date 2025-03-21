export default {
  async fetch(request, env) {
    return await handleRequest(request, env);
  }
};

async function handleRequest(request, env) {
  const url = new URL(request.url);
  const authHeader = request.headers.get("Authorization");
  const queryApiKey = url.searchParams.get("api_key");

  // 从环境变量读取敏感信息 ⚠️ 需在 Workers 控制台配置这些变量
  const SECRET_KEY = env.SECRET_KEY;
  const TG_BOT_TOKEN = env.TELEGRAM_BOT_TOKEN;
  const TG_CHAT_ID = env.TELEGRAM_CHAT_ID;

  // 认证失败响应
  const unauthorizedResponse = new Response(JSON.stringify({ error: "Unauthorized" }), { status: 403 });

  // ✅ 处理 POST 请求
  if (request.method === "POST") {
    if (!authHeader || authHeader !== `Bearer ${SECRET_KEY}`) {
      return unauthorizedResponse;
    }

    const data = await request.json();
    if (!data.ip || !data.port || !data.method || !data.password) {
      return new Response(JSON.stringify({ error: "Invalid input, missing fields" }), { status: 400 });
    }

    // 生成并存储订阅
    const ssNode = `ss://${btoa(`${data.method}:${data.password}@${data.ip}:${data.port}`)}#GoHomeEasy`;
    await env.KV_NAMESPACE.put("latest_subscription", ssNode);

    // 🔔 新增电报推送功能
    if (TG_BOT_TOKEN && TG_CHAT_ID) {
      try {
        const telegramUrl = `https://api.telegram.org/bot${TG_BOT_TOKEN}/sendMessage`;
        const message = `🎉 节点已更新\nIP: ${data.ip}\n端口: ${data.port}\n加密: ${data.method}`;
        
        await fetch(telegramUrl, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            chat_id: TG_CHAT_ID,
            text: message,
            disable_notification: false
          })
        });
      } catch (error) {
        console.error("Telegram推送失败:", error);
      }
    }

    return new Response(JSON.stringify({ message: "Node updated successfully" }), { status: 200 });
  }

  // ✅ 处理 GET 请求（保持不变）
  if (request.method === "GET") {
    if (!queryApiKey || queryApiKey !== SECRET_KEY) {
      return unauthorizedResponse;
    }

    const subscription = await env.KV_NAMESPACE.get("latest_subscription");
    return subscription 
      ? new Response(btoa(subscription), { headers: { "Content-Type": "text/plain" } })
      : new Response("", { status: 204 });
  }

  return new Response("Invalid request", { status: 405 });
}
