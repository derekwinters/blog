## Blog Posts

<table border=0>
  {% for post in site.posts %}
  <tr>
    <td>{{ post.date | date: "%Y-%m-%d" }}</td>
    <td><a href="{{ post.url }}">{{ post.title }}</a></td>
  </tr>
  {% endfor %}
</table>
