---
layout: default
title: Portfolio
permalink: /portfolio/
---
<div class="home">
  {% if site.data.universityAssignments.size > 0 %}
    <div class="list-group">
      <div class="container-fluid">
        
        <div class="row">
          {% for assignment in site.data.universityAssignments %}
            <div class="col-md-6 project">
              <a href="/assignment/{{ assignment.slug }}/">
                <div class="media-body">
                  <h4 class="media-heading">{{ assignment.title }}</h4>
                </div>
              </a>
              {% if assignment.subtitle %}<p>{{ assignment.subtitle }}</p>{% endif %}
            </div>
          {% endfor %}
        </div>
      </div>
    </div>
  {% else %}
  {% endif %}
</div>
