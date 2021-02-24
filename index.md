## Blog Posts

<table border=0>
  {% for post in site.posts %}
  <tr>
    <td>{{ post.date }}</td>
    <td><a href="{{ post.url }}">{{ post.title }}</a>
  </tr>
  {% endfor %}
</table>

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
