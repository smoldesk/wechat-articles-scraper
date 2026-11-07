# WeChat Articles Scraper

![WeChat Articles Scraper: 公众号 articles with full text, images and account details without login](https://raw.githubusercontent.com/smoldesk/apify-scrapers/main/assets/wechat/banner.webp)

Scrape WeChat Official Account (微信公众号) articles without a WeChat account, login, cookies or API key. Search any keyword and get each article's full text as Markdown and plain text, all images, title, publish time, account name and ID (gh_…), author, the original/repost flag (原创) and the IP location WeChat shows. Runs on [Apify](https://apify.com/smoldesk/wechat-articles-scraper).

This repo has usage examples only. The scraper runs on Apify, so there's nothing to install.

## What you get

- **Keyword search** through Sogou WeChat search (搜狗微信), up to 30 articles per keyword and about 200 per run.
- **Full text** of every article as `markdown` (headings, lists, images) and `text`, ready for LLMs, translation and sentiment.
- **Account details**: name, `gh_` ID, biz, description, avatar.
- **Article details**: author, publish time, cover, images, original or repost, IP location, the "阅读原文" source link. Image posts and video posts are handled too.
- **Article URLs mode**: paste `mp.weixin.qq.com/s/…` links to get their full text.
- **Only new articles**: schedule it and each run returns and charges only articles it hasn't seen before.
- **Date range**: keep only articles from a period.

## Run it

Open the [WeChat Articles Scraper on Apify](https://apify.com/smoldesk/wechat-articles-scraper), add keywords and click Start. Or call it from code with your [Apify token](https://console.apify.com/settings/integrations).

### curl

```bash
curl -X POST "https://api.apify.com/v2/acts/smoldesk~wechat-articles-scraper/run-sync-get-dataset-items?token=YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "searchQueries": ["新能源汽车", "比亚迪 海外"],
    "maxResultsPerSearch": 20,
    "includeFullText": true,
    "postedAfter": "2026-10-01"
  }'
```

### Python

```bash
pip install apify-client
```

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_TOKEN")
run = client.actor("smoldesk/wechat-articles-scraper").call(run_input={
    "searchQueries": ["瑞幸咖啡"],
    "maxResultsPerSearch": 30,
    "onlyNewResults": True,
})
for article in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(article["publishedAt"], article["accountName"], article["title"])
```

### JavaScript / Node.js

```bash
npm install apify-client
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: 'YOUR_TOKEN' });
const run = await client.actor('smoldesk/wechat-articles-scraper').call({
    articleUrls: ['https://mp.weixin.qq.com/s/XXXXXXXXXXXXXXXXXXXXXX'],
});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const a of items) console.log(a.title, a.textLength);
```

### LangChain

```bash
pip install langchain-apify
```

```python
from langchain_apify import ApifyWrapper
from langchain_core.documents import Document

apify = ApifyWrapper()  # reads APIFY_API_TOKEN
loader = apify.call_actor(
    actor_id="smoldesk/wechat-articles-scraper",
    run_input={"searchQueries": ["新能源汽车"], "maxResultsPerSearch": 30},
    dataset_mapping_function=lambda v: Document(
        page_content=v.get("markdown") or v.get("summary") or v.get("title") or "",
        metadata={k: v.get(k) for k in ("title", "accountName", "publishedAt", "url", "articleId", "ipLocation")},
    ),
)
docs = loader.load()
```

LlamaIndex: `ApifyActor` from `llama-index-readers-apify`. Claude, CrewAI and other MCP clients: add `https://mcp.apify.com?actors=smoldesk/wechat-articles-scraper` as an MCP server.

## Sample output

```json
{
  "type": "article",
  "title": "国外直升机人工智能技术应用现状及未来发展趋势",
  "accountName": "战略前沿技术",
  "accountId": "gh_3d97b30e7f44",
  "author": "直升机所 宋烨",
  "publishedAt": "2026-11-04T15:30:58+00:00",
  "summary": "持续推进人工智能(AI)在直升机领域落地应用,已经从早期实验室算法验证,过渡到实机试飞…",
  "isOriginal": false,
  "ipLocation": "北京",
  "contentType": "article",
  "contentState": "ok",
  "articleId": "MzI3MTkwMzM0Nw==_2248073827_5",
  "searchQuery": "人工智能",
  "searchRank": 1,
  "markdown": "…",
  "text": "…",
  "textLength": 9723,
  "images": ["https://mmbiz.qpic.cn/…"]
}
```

Export as JSON, CSV or Excel, or send it to Google Sheets, Slack, Make, Zapier or n8n.

## Pricing

Pay per article. No monthly fee. Each article is charged once.

| Result | Per 1,000 |
|---|---|
| Article with full text | $8 |
| Search card only (full text off, or the article couldn't be opened) | $2 |

Duplicates, articles outside your date range and articles skipped by Only new articles are not charged.

## Limits

- Up to 30 articles per keyword and about 200 per run, which is what Sogou opens to a visitor without a login. Use more specific keywords and a daily schedule to cover more.
- No read counts, likes or comments. WeChat only shows those inside the app to logged-in users.
- No account search or full account history. Sogou has switched account search off.
- Links from Sogou are signed temporary links. Keep `articleId` as your stable key.

## FAQ

**Does WeChat have an API for Official Account articles?**
Not for searching other people's articles. This scraper uses Sogou WeChat search and the public article pages on mp.weixin.qq.com.

**Do I need a WeChat account, a Chinese phone number or a proxy?**
No. Nothing to log in to and nothing to configure.

**How do I monitor a brand on WeChat?**
Start from the ready-made task [Monitor a brand on WeChat](https://apify.com/smoldesk/wechat-articles-scraper/examples/wechat-brand-monitoring), save it and add a daily schedule.

**Can I also track Weibo and Toutiao?**
Yes, with the [Weibo Scraper](https://github.com/smoldesk/weibo-scraper) and [Toutiao Scraper](https://github.com/smoldesk/toutiao-scraper).

## More scrapers

WeChat Articles Scraper is one of 20 no-login scrapers by Smoldesk Studios. See the full list: [smoldesk/apify-scrapers](https://github.com/smoldesk/apify-scrapers).
