---
# 这里故意不写 title。
# 主题模板会把「页面标题 | 站点标题」拼起来，首页两者相同会让 <title> 重复；
# 留空后首页标题直接取站点名（og:title 同样回落到站点名）。
#
# 首页正文已不再用这个文件的 Markdown：layouts/index.html 覆盖了主题首页模板，
# 改为「每日一句 + 最新文章 + 分类入口」三块。
#   名句内容  → data/quotes.yaml
#   首页样式  → assets/css/home.css
#   首页文案  → i18n/zh-cn.toml 里的 home_* 键
# 所以下面不再保留任何正文（留着也不会被渲染）。
---
