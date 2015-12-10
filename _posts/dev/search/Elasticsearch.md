title: MAC elasticsearch
date: 2015-12-8 10:19:56
tags: elasticsearch
category: dev
---

1. 安装
```bash
brew install elasticsearch
```

2. Monitor Elasticsearch
```bash
# Step 1: Install Marvel into Elasticsearch:  
bin/plugin install license
bin/plugin install marvel-agent
# Step 2: Install Marvel into Kibana  
bin/kibana plugin --install elasticsearch/marvel/latest
# Step 3: Start Elasticsearch and Kibana  bin/elasticsearch
bin/kibana
# Step 4: Navigate to http://localhost:5601/app/marvel
```