---
layout:     post
title:      Web3 身份验证一条龙：NextAuth + WAGMI + SIWE 实战指南
date:       2025-09-05
header-img: img/post-bg-desk.jpg
catalog: true
---

关键词：**Web3 身份验证** · **SIWE** · **Sign-In with Ethereum** · **WAGMI** · **NextAuth** · **RainbowKit** · **next-auth** · **区块链钱包登录** · **Next.js SIWE** · **去中心化登录**

---

## 什么是 Web3 身份验证？
在传统互联网里，你需要邮箱+密码；到了 Web3，用户只需**区块链钱包地址**——比如 MetaMask、Trust Wallet 或 WalletConnect 所列钱包之一——就能登录去中心化应用（dApp）。整个过程**不存储密码**，无需信任中心化服务，天然避免撞库与拖库风险。  
👉 [5 分钟实现无密码登录，立即体验一键链上身份！](https://okxdog.com/)

---

## 为何选择 SIWE 标准做 Web3 登录？
Sign-In with Ethereum（简称 **SIWE**）最早由 Spruce 推出，现已被 W3C 收录为标准草案。它让用户体验“**一个签名人机对话**”——前端生成并展示消息，用户动动手指即可完成签名，后端只需验证签名即可确认其链上身份。优势总结：

1. **加密级安全**：私钥只留在用户本地，签名不可逆推密钥。
2. **无密码 UX**：告别验证码、多因素输入。
3. **可扩展授权**：同一消息可附带权限范围、过期时间，调用智能合约一步到位。

---

## 工具体系一览
要在 Next.js 中落地 Web3 身份验证，下列库各司其职、默契配合：

- **WAGMI**：React Hooks 一步到位搞定钱包连接、链切换、消息签名。
- **viem**：新一代 TypeScript 链交互库，打包 Gas 优化与 ABI 解析。
- **SIWE** 官方包：生成与验证规范化消息，兼容多链。
- **Iron Session**：Next.js 场景下轻量级 cookie-session 方案，防 CSRF、支持 server component。
- **NextAuth**：即插即用的认证后端，支持 OAuth、Credentials、Providerless。
- **RainbowKit**：高颜值钱包连接弹窗，多链、主题皆可定制。

---

## 环境搭建：三步起跑
1. 机器配置：`pnpm` + Node 18+。
2. 初始化项目  
   ```bash
   pnpm create next-app web3-auth
   cd web3-auth
   pnpm add wagmi viem@^2 @tanstack/react-query \
          next-auth iron-session @rainbow-me/rainbowkit
   ```
3. 目录结构（后续所有代码皆基于此）  
   ```
   web3-auth/
     └─ src/
        ├─ app/
        │  └─ api/
        │     └─ auth/[...nextauth]/route.ts
        │     └─ nonce/route.ts
        ├─ providers/
        │  └─ Providers.tsx
        │  └─ ReactQueryProvider.tsx
        │  └─ RainbowKitProvider.tsx
        │  └─ AuthProvider.tsx
        ├─ utils/
        │  └─ session.ts
        │  └─ serverActions.ts
        │  └─ authenticationAdapter.ts
   ```

---

## Provider 全家桶：层层递进
### ReactQueryProvider
```ts
"use client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactNode, useState } from "react";

export default function ReactQueryProvider({ children }: { children: ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: { queries: { staleTime: 60 * 1000 } },
      })
  );
  return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>;
}
```

### AuthProvider：让 _next-auth_ 的 Session 在 server component 中也可用  
```ts
"use client";
import { SessionProvider } from "next-auth/react";
import { ReactNode } from "react";

export default function AuthProvider({
  children,
  session,
}: {
  children: ReactNode;
  session: any;
}) {
  return <SessionProvider session={session}>{children}</SessionProvider>;
}
```

### RainbowKitProvider：粘合 RainbowKit、WAGMI 与 NextAuth
```ts
"use client";
import "@rainbow-me/rainbowkit/styles.css";
import { cookieToInitialState, WagmiProvider, createConfig } from "wagmi";
import { RainbowKitProvider, RainbowKitAuthenticationProvider } from "@rainbow-me/rainbowkit";
import { useSession } from "next-auth/react";
import React from "react";
import { authenticationAdapter } from "@/utils/authenticationAdapter";
import ReactQueryProvider from "./ReactQueryProvider";

const wagmiConfig = createConfig(/* 你的配置，此处略 */);
export default function RKProvider({ children, cookie }: { children: React.ReactNode; cookie: string }) {
  const { status } = useSession();
  const initialState = cookieToInitialState(wagmiConfig, cookie);
  return (
    <WagmiProvider config={wagmiConfig} initialState={initialState}>
      <ReactQueryProvider>
        <RainbowKitAuthenticationProvider adapter={authenticationAdapter} status={status}>
          <RainbowKitProvider>{children}</RainbowKitProvider>
        </RainbowKitAuthenticationProvider>
      </ReactQueryProvider>
    </WagmiProvider>
  );
}
```

---

## Server 层：Iron Session 统一托管
所有需要共享的临时数据（比如随机 nonce）丢进 **server-iron-session**：

```ts
// utils/session.ts
import { IronSessionOptions, getIronSession } from "iron-session";
import { cookies } from "next/headers";

export const sessionOptions: IronSessionOptions = {
  password: process.env.IRON_PASSWORD as string,
  cookieName: "siwe_session",
  cookieOptions: { secure: process.env.NODE_ENV === "production" },
};

export async function getServerSession() {
  return getIronSession(await cookies(), sessionOptions);
}
```

---

## Server Actions：证书式共享逻辑
```ts
// utils/serverActions.ts
"use server";
import { getServerSession } from "@/utils/session";

export const storeNonce = async (nonce: string) => {
  const session = await getServerSession();
  session.nonce = nonce;
  await session.save();
};

export const getNonce = async (): Promise<string> => {
  const session = await getServerSession();
  return session.nonce || "";
};
```

---

## 前端身份适配器：一行签名搞定
```ts
// utils/authenticationAdapter.ts
import { createAuthenticationAdapter } from "@rainbow-me/rainbowkit";
import { createSiweMessage } from "viem/siwe";
import { signIn, signOut } from "next-auth/react";

export const authenticationAdapter = createAuthenticationAdapter({
  getNonce: async () => {
    const res = await fetch("/api/nonce", { method: "POST" });
    const { nonce } = await res.json();
    return nonce;
  },
  createMessage: ({ nonce, address, chainId }) =>
    createSiweMessage({
      domain: window.location.host,
      address,
      statement: "Sign in with Ethereum to this dApp.",
      uri: window.location.origin,
      version: "1",
      chainId,
      nonce,
    }),
  verify: async ({ message, signature }) => {
    const res = await signIn("credentials", { redirect: false, message, signature });
    return Boolean(res?.ok);
  },
  signOut: async () => signOut(),
});
```

---

## API Route：两个端点就够
### `/api/nonce`（生成 11 位随机数并落库）
```ts
import { NextResponse } from "next/server";
import { generateNonce } from "siwe";
import { storeNonce } from "@/utils/serverActions";

export async function POST() {
  const nonce = generateNonce();
  await storeNonce(nonce);
  return NextResponse.json({ nonce }, { status: 201 });
}
```

### `/api/auth/[...nextauth]/route`（NextAuth 自动路由）
```ts
export { authConfig as GET, authConfig as POST } from "./authOptions";
```
其中 `authOptions` 的精简核心：
```ts
import CredentialsProvider from "next-auth/providers/credentials";
import { SiweMessage } from "siwe";
import { getNonce } from "@/utils/serverActions";

export const authConfig = {
  providers: [
    CredentialsProvider({
      credentials: { message: {}, signature: {} },
      authorize: async (credentials) => {
        const siwe = new SiweMessage(credentials!.message);
        const serverNonce = await getNonce();
        if (siwe.nonce !== serverNonce) throw new Error("nonce 不匹配");

        const { success } = await siwe.verify({
          signature: credentials!.signature,
          domain: siwe.domain,
          nonce: serverNonce,
        });
        if (!success) return null;

        return {
          id: siwe.address,
          walletAddress: siwe.address,
          accessToken: "FUTURE_REPLACE_WITH_JWT",
        };
      },
    }),
  ],
  callbacks: {
    jwt: ({ token, user }) => (user ? { ...token, ...user } : token),
    session: ({ session, token }) => ({
      ...session,
      user: { ...session.user, walletAddress: token.walletAddress },
      accessToken: token.accessToken,
    }),
  },
};
```

---

## FAQ：最常见的 5 个疑问  

**Q1：如何防止重放攻击？**  
A：每次登录会生成新的 `nonce`，并写入服务器 session，重复验证时会检测 nonce 存在且被一次性消耗。

**Q2：SIWE 消息里一定要包含 chainId 吗？**  
A：强烈推荐。把 `chainId` 写进消息体，可避免用户跨链重放，同时也方便在服务端核对当前链。

**Q3：Iron Session 的安全性够得上生产吗？**  
A：Iron Session 基于加密 cookie，采用 AES-256-GCM + 密钥轮换。只要 `IRON_PASSWORD` ≥32 字符，且仅存在于服务端，就能通过专业审计。

**Q4：RainbowKit 能减小 bundle size 吗？**  
A：v2 起已支持 Tree-shaking，配合 `next/dynamic` 导入，可让首屏仅增加 ~5 KB。

**Q5：前端 URI 变动会不会导致验证失败？**  
A：会。务必保证 `domain` 与 `uri` 字段落在同一 Origin，否则签名检验会抛出 `Invalid domain` 的错误。

---

## 实战编年史：15 分钟上线
1. `pnpm dev`：浏览器闪现 RainbowKit 弹窗。  
2. 连接钱包 → 签名 → 地址闪现导航栏。  
3. 刷新页面 → 仍保持登录，说明 Session+JWT 已生效。

👉 [一键 Fork 开源模板，立即把 Web3 登录搬进你的 DApp！](https://okxdog.com/)

---

## 总结与下一步
通过组合 **SIWE + WAGMI + NextAuth + RainbowKit**，你将传统“注册/登录”压缩到了几秒钟的签名体验，并且获得了可自定义权限、链上可验证的身份系统。下一步可探索：

- 用 **ZKP** 增强隐私，隐藏实地址。
- 接入 **OAuth + SIWE 二级登录**，为 Web2 用户铺设丝滑迁移路径。
- 将 Session 与 **on-chain attestations** 打通，实现链上 KYC、徽章系统。