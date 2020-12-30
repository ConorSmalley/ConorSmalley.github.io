---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
---
<h2 class="post-list-heading">Posts</h2>

{% for post in site.posts %}
<div>
    <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
    <h2>
    <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
    </h2>
    {{post.excerpt}}
</div>
{% endfor %}