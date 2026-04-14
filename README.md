[中文文档](https://github.com/MiceXH/GSC-Saver-Doc/blob/main/README_CN.md) 

# GSC Saver: Permanent Backup Solution for Google Search Console Data

Google Search Console only keeps your search data for **16 months**. If you don't archive it, you lose your historical SEO insights forever.

This repository provides two ways to solve this:

1. **A Free Script:** For developers who want to manually handle the GSC API.
2. **GSC Saver Extension:** A "Buy-it-once" pro tool for automated, privacy-first backup to Google Drive.

------

## 🛑 The Problem: The "16-Month Data Cliff"

Google automatically deletes your keyword, click, and impression data after 16 months.

- **No YoY Analysis:** You can't compare this year's performance with data from 2 years ago.
- **Expensive Solutions:** Enterprise SEO suites charge **$100+/month** just to "archive" your data.
- **Data Ownership:** Your SEO data shouldn't have an expiration date.

------

## 🛠 The Free Solution: Manual Backup Script

If you are comfortable with Script and API configurations, you can use the legacy script included in this repo.

- **Features:** Connects to GSC API, fetches data, and saves to Google Sheet.
- **Documentation:** [View Script Manual (中文)](https://github.com/MiceXH/GSC-Saver-Doc/blob/main/GSC%E6%95%B0%E6%8D%AE%E5%A4%87%E4%BB%BD%E6%93%8D%E4%BD%9C%E6%89%8B%E5%86%8C.md)  [View Script Manual](https://github.com/MiceXH/GSC-Saver-Doc/blob/main/GSC%E6%95%B0%E6%8D%AE%E5%A4%87%E4%BB%BD%E6%93%8D%E4%BD%9C%E6%89%8B%E5%86%8C.md)
- **Limitations:** Requires technical setup, manual execution, and lacks automated Google Drive syncing.

------

## 🚀 The Pro Solution: GSC Saver Extension

**GSC Saver** is a Chrome Extension built for site owners and SEO professionals who want a "set it and forget it" solution.

### Key Features:

- **Zero Configuration:** No API keys or coding required.
- **Automated Sync:** Directly bridges GSC data to **your own Google Drive**.
- **Data Ownership:** No 3rd-party servers. Your sensitive SEO data stays between GSC and your Google Drive.
- **Buy-it-once:** Stop the subscription fatigue. One payment, lifetime protection.

### [👉 Get GSC Saver on Chrome Web Store](https://chromewebstore.google.com/detail/jafnmmnidbecklolnjlkakhppjmoolfn?utm_source=item-share-cb)

------

## 📊 Comparison: Script vs. Extension

| **Feature**            | **Open Source Script (Free)** | **GSC Saver Extension (Pro)** |
| ---------------------- | ----------------------------- | ----------------------------- |
| **Setup Time**         | 30-60 mins (Technical)        | **< 1 min**                   |
| **Storage**            | Local CSV files               | **Google Drive (Auto-sync)**  |
| **Automation**         | Manual / Task Scheduler       | **Full Background Sync**      |
| **Multi-site Support** | Manual per site               | **1-Click Bulk Sync**         |
| **Maintenance**        | You maintain the code         | **Lifetime Updates**          |
| **Price**              | Free                          | **$29 (One-time Payment)**    |

------

## 🔒 Privacy & Security

We believe you should own your data.

- **Local Processing:** The extension processes data locally in your browser.
- **Direct Bridge:** Data is transferred directly from Google Search Console API to Google Drive.
- **Zero-Knowledge:** We never see your keywords, clicks, or site URLs.

------

## 📖 How to Use (Extension)

1. Install **GSC Saver** from the Chrome Web Store.
2. Click the GSC Saver icon and authorize your Google Drive.
3. Turn on Auto Sync.
4. Sit back and watch your SEO assets grow in your GDrive.

------

## 👨‍💻 Why I Built This

I'm an independent developer and SEO enthusiast. I got tired of paying monthly subscriptions for simple utility tools. I built GSC Saver to provide a privacy-first, affordable alternative for the indie maker community.

If this tool saves you a headache, a **Star ⭐** to this repo would be much appreciated!

------

## 🔗 Links

- **Official Website:** [https://www.gscsaver.com/]
- **Support & FAQ:** [maxinhao1997@gmail.com]
- **Twitter:** [@Mice](https://x.com/Mice242697)