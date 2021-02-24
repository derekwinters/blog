## Blog Posts

<table border=0>
  {% for post in site.posts %}
  <tr>
    <td style="border:none">{{ post.date | date: "%Y-%m-%d" }}</td>
    <td style="border:none"><a href="{{ post.url }}">{{ post.title }}</a></td>
  </tr>
  {% endfor %}
</table>
