<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# My Firecrawl self-hosted leads to crashing my Streamlet application and high CPU load. Can the Firecrawl be set that it uses the CPU or LLMs for crawling web pages

To address high CPU load and application crashes in your self-hosted Firecrawl instance, consider these optimizations based on reported configurations and developer experiences:

### **Resource Management Adjustments**

1. **Worker Concurrency Control**
Limit simultaneous tasks using the `NUM_WORKERS_PER_QUEUE` environment variable. For example:

```env
NUM_WORKERS_PER_QUEUE=2  # Reduce from default values
```

This prevents overloading your CPU by capping parallel processing[^2][^7].
2. **Docker Resource Constraints**
Add CPU/memory limits in your `docker-compose.yaml`:

```yaml
services:
  api:
    deploy:
      resources:
        limits:
          cpus: '1.5'
          memory: 2G
```

This ensures Firecrawl doesn’t monopolize system resources[^2][^4].

---

### **LLM Configuration (If Using Local Models)**

- **Switch to Lightweight Models**
Replace resource-heavy models like `gpt-4o` with smaller alternatives in your `.env`:

```env
MODEL_NAME=deepseek-r1:8b  # From search results[^1]
MODEL_EMBEDDING_NAME=nomic-embed-text
```

- **Disable Unnecessary Features**
Comment out unused services like Playwright if not needed



---

### **Crawling Efficiency**

- **Batch Processing**
Use asynchronous batch scraping to avoid blocking resources:

```python
from firecrawl import FirecrawlApp
app = FirecrawlApp()
job = app.async_batch_scrape_urls(url_list)  # Processes URLs in background[^5]
```

- **Timeout Configuration**
Add `timeout` parameters to prevent hung tasks:

```python
app.scrape_url(url, params={"timeout": 30})  # 30-second timeout[^8]
```


---

### **Troubleshooting Crashes**

1. **Monitor Resource Usage**
Enable detailed logging to identify bottlenecks:

```env
LOGGING_LEVEL=DEBUG  # Track CPU/memory spikes[^1][^2]
```

2. **Validate Ollama Integration**
Set the project to local LLMs, ensure proper configuration:

```env

I did set this and its endpoint is tested:
OLLAMA_BASE_URL=http://host.docker.internal:11434/api  # Verify port/network access[^1]
```


---

### **Alternative: Cloud vs. Self-Hosted**

Developers report **50x faster performance** and **58% lower costs** when using Firecrawl’s managed cloud service compared to self-hosted setups[^5]. Consider migrating if resource constraints persist.

By implementing these changes, you should see reduced CPU load and improved stability. Most issues stem from unbounded concurrency and unoptimized model choices rather than inherent Firecrawl limitations[^2][^5][^7].

<div style="text-align: center">⁂</div>

[^1]: https://github.com/mendableai/firecrawl/issues/1467

[^2]: https://github.com/mendableai/firecrawl/issues/722

[^3]: https://www.blott.studio/blog/post/how-to-build-llm-ready-datasets-with-firecrawl-a-developers-guide

[^4]: https://github.com/mendableai/firecrawl/issues/467

[^5]: https://www.blott.studio/blog/post/how-firecrawl-cuts-web-scraping-time-by-60-real-developer-results

[^6]: https://python.langchain.com/docs/integrations/document_loaders/firecrawl/

[^7]: https://www.pondhouse-data.com/blog/crawl-websites-for-llms

[^8]: https://docs.crawl4ai.com/core/browser-crawler-config/

[^9]: https://docs.camel-ai.org/cookbooks/ingest_data_from_websites_with_Firecrawl.html

[^10]: https://docs.firecrawl.dev/contributing/self-host

[^11]: https://github.com/mendableai/firecrawl/issues/1307

[^12]: https://www.reddit.com/r/LocalLLaMA/comments/1eiasxt/launch_yc_firecrawl_open_source_crawling_and/

[^13]: https://github.com/mendableai/firecrawl/issues

[^14]: https://github.com/mendableai/firecrawl/issues/1300

[^15]: https://www.firecrawl.dev

[^16]: https://github.com/devflowinc/firecrawl-simple

[^17]: https://www.firecrawl.dev/blog/mastering-firecrawl-scrape-endpoint

[^18]: https://discuss.streamlit.io/t/streamlit-app-crashing-after-few-minutes-of-usage/40803

[^19]: https://discuss.streamlit.io/t/app-crashing-even-before-using-it-resources-limit/19239

[^20]: https://community.ipfire.org/t/cpu-usage-increase/13465

[^21]: https://apify.com/apify/website-content-crawler/issues/poor-cpu-utilization-ccLjgv0wfXuXWeXtv

