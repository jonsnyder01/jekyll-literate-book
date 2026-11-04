---
title: "Layouts and includes"
entangled:
  version: "2.5"
  namespace: layouts
---

Every page of a book ends with links to the pages before and after it in reading
order. The weave sets `page.previous` and `page.next` on each one: the title
page leads to the table of contents, and that to the first chapter.

```html
<!--| file: gem/_includes/reading-order.html
<nav class="reading-order">
  {% if page.previous %}<a class="previous" href="{{ page.previous.url | relative_url }}">← {{ page.previous.title }}</a>{% endif %}
  {% if page.next %}<a class="next" href="{{ page.next.url | relative_url }}">{{ page.next.title }} →</a>{% endif %}
</nav>
```
