---
layout: page
permalink: /teaching/
title: teaching
description: Material for courses and workshops
nav: true
---

<div class="courses">
  {% if site.teaching  %}
    <div class="table-responsive">
      <table class="table table-sm">
      {% assign courses = site.teaching | reverse %}
      {% for item in courses %}
        <tr>
          <th scope="row">{{ item.date | date: "%b %-d, %Y" }}</th>
            {% if item.inline %}
            <td>
              {{ item.title | remove: '<p>' | remove: '</p>' | emojify }}
            </td>
            <td>
              {% if item.slides %}
                <a class="badge teaching-button waves-effect center font-weight-light mr-1" href="{{ item.slides | prepend: '/assets/pdf/' | prepend: site.baseurl | prepend: site.url }}" target="_blank">Slides</a>
              {% endif %}
              {% if item.html %}
                <a class="badge teaching-button waves-effect center font-weight-light mr-1" href="{{ item.html }}" target="_blank">Material</a>
              {% endif %}
            </td>
            {% else %}
              <td>
              {{ item.title | remove: '<p>' | remove: '</p>' | emojify }}
              </td>
              <td>                
                <a class="badge course-title teaching-button waves-effect center font-weight-light mr-1" href="{{ item.url | relative_url }}">Material</a>            
            </td>
            {% endif %}
        </tr>
      {% endfor %}
      </table>
    </div>
  {% else %}
    <p>No courses so far...</p>
  {% endif %}
</div>

