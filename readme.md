# 🌟 Quit Habit App (健康习惯助手)

> **A habit tracker that truly understands human nature.**  
> **一款真正懂人性的习惯养成助手。**

<p align="center">
  <img src="static/logo.png" width="100" />
</p>

---

## 📖 Introduction (简介)

This is a pure local, privacy-first, and highly customized habit tracking application built with **UniApp (Vue 3)**. It features a unique "Dual-Track" mechanism designed to help users quit bad habits strictly while cultivating good ones leniently.

这是一个基于 **UniApp (Vue 3)** 开发的纯本地、隐私优先、高度定制化的习惯追踪应用。它独创了“双轨制”机制：用严苛的逻辑帮你戒除恶习，用宽容的态度陪你养成好习惯。

### ✨ Core Features (核心亮点)

*   **☯️ Dual-Track System (双轨制)**:
    *   **Strict Mode (戒除)**: Resets streak to ZERO if you miss a day. No mercy. (断签即归零，严防破戒)
    *   **Lenient Mode (养成)**: Accumulates total days. Never resets. (累计天数，宽容鼓励)
*   **📅 Native Calendar Integration (系统日历提醒)**:
    *   Uses `Native.js` to write reminders directly into the Android System Calendar. 100% delivery rate even if the App is killed. (直接写入系统日历，无视杀后台，提醒到达率 100%)
*   **🛡️ Anti-Cheat System (防作弊)**:
    *   Prevents time-travel cheating. Future data will be masked, and back-dating is blocked. (防止改时间作弊，未来数据自动屏蔽，回档补签会被拦截)
*   **💾 Pure Local Data (纯本地)**:
    *   No server, no login, no privacy leakage. Support JSON export/import via clipboard. (无服务器，无登录，不上传隐私。支持剪贴板导入导出存档)

---

## 🛠️ Tech Stack (技术栈)

*   **Framework**: [UniApp](https://uniapp.dcloud.io/) (Vue 3)
*   **Build Tool**: Vite
*   **Storage**: LocalStorage (uni.setStorageSync)
*   **Native Capability**: Android Native.js (Calendar Provider, Permissions)

---

## 👨‍💻 Author (作者)

*   **Name**: 道系青年Lee (Daoist Youth Lee)
*   **Bilibili**: [b站up主 道系青年Lee](https://space.bilibili.com/437113079)
*   **Philosophy**: "Technology should serve humanity, not control it."

> I am not a CS major, but with the help of AI, I brought my product philosophy to life. This project proves that **ideas matter more than syntax**.
> 
> 我并非计算机科班出身，但在 AI 的协助下，我将我的产品理念变成了现实。这证明了：**想法比语法更重要。**


## ☕ Support Me (请我喝咖啡)

If this app helps you become a better version of yourself, you can buy me a coffee to keep my keyboard firing!  
如果这个 App 帮到了你，或者你喜欢我的开发理念，欢迎打赏支持，让我的AI运行速度更快一点！

<p align="center">
  <img src="static/reward.jpg" width="200" alt="Alipay Reward Code" />
</p>

> **Note**: This is purely voluntary. The App will always be free and open source.  
> **注**：打赏纯属自愿。App 永远免费开源，绝无付费墙。


---

## ⚖️ License (开源协议)

**GNU General Public License v3.0 (GPL-3.0)**

You are free to use, modify, and distribute this software, BUT you must comply with the following conditions:
您可以自由使用、修改和分发本软件，但必须遵守以下条件：

1.  **Open Source (必须开源)**: If you modify this code and release it, your version **MUST also be open source** under GPL-3.0. (如果您修改代码并发布，您的版本也必须开源)
2.  **Attribution (署名)**: You must clearly state the original author is **"b站up主 道系青年Lee"**. (必须保留原作者署名)
3.  **No Commercial Use (严禁商用)**: This project is for educational and personal use only. Selling this App or embedding ads is strictly prohibited. (严禁将本项目打包收费或植入广告牟利)

---

## 🚀 How to Run (如何运行)

1.  Download [HBuilderX](https://www.dcloud.io/hbuilderx.html).
2.  Import this project folder.
3.  Run -> Run to App Base (Android) or Browser.
4.  **Note**: Calendar features require a custom debugging base or a real device build.

---

<p align="center">
  Made with AI by <a href="https://space.bilibili.com/437113079">道系青年Lee</a>

</p>

