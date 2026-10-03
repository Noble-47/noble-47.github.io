---
layout: index
title: Project Hub
---


{% comment %} 1. Extract all unique tags across projects {% endcomment %}
{% assign project_pages = site.pages | where_exp: "item", "item.path contains 'projects/'" | sort: "date" | reverse %}
{% assign all_tags = "" | split: "" %}

{% for project in project_pages %}
  {% if project.name != "index.md" and project.title and project.tags %}
    {% assign project_tags = project.tags | join: " " | split: " " %}
    {% assign all_tags = all_tags | concat: project_tags | uniq %}
  {% endif %}
{% endfor %}

<!-- 2. Tag Filter UI Bar -->
<div class="tag-filter-bar">
  <button class="filter-btn active" data-tag="all">All</button>
  {% for tag in all_tags %}
    <button class="filter-btn" data-tag="{{ tag | slugify }}">{{ tag }}</button>
  {% endfor %}
</div>
<br>

<!-- 3. Projects Container -->
<div class="projects-list">
  {% for project in project_pages %}
    {% if project.name != "index.md" and project.title %}
      {% assign tag_slugs = "" | split: "" %}
      {% if project.tags %}
        {% assign raw_tags = project.tags | join: " " | split: " " %}
        {% for t in raw_tags %}
          {% assign slug = t | slugify %}
          {% assign tag_slugs = tag_slugs | push: slug %}
        {% endfor %}
      {% endif %}

      <article class="project-item" data-tags="{{ tag_slugs | join: ' ' }}">
        <div class="project-header-row">
          <h2 class="project-title">
            <a href="{{ project.url | relative_url }}">{{ project.title }}</a>
          </h2>

          <div class="project-links">
            {% if project.repository_url %}
              <a href="{{ project.repository_url }}" target="_blank" rel="noopener noreferrer" class="project-link-icon" title="View Code on GitHub">
                <svg viewBox="0 0 16 16" width="20" height="20" fill="currentColor">
                  <path d="M8 0c4.42 0 8 3.58 8 8a8.013 8.013 0 0 1-5.45 7.59c-.4.08-.55-.17-.55-.38 0-.27.01-1.13.01-2.2 0-.75-.25-1.23-.54-1.48 1.78-.2 3.65-.88 3.65-3.95 0-.88-.31-1.59-.82-2.15.08-.2.36-1.02-.08-2.12 0 0-.67-.22-2.2.82-.64-.18-1.32-.27-2-.27-.68 0-1.36.09-2 .27-1.53-1.03-2.2-.82-2.2-.82-.44 1.1-.16 1.92-.08 2.12-.51.56-.82 1.28-.82 2.15 0 3.06 1.86 3.75 3.64 3.95-.23.2-.44.55-.51 1.07-.46.21-1.61.55-2.33-.66-.15-.24-.6-.83-1.23-.82-.67.01-.27.38.01.53.34.19.73.9.82 1.13.16.45.68 1.31 2.69.94 0 .67.01 1.3.01 1.49 0 .21-.15.45-.55.38A7.995 7.995 0 0 1 0 8c0-4.42 3.58-8 8-8Z"></path>
                </svg>
              </a>
            {% endif %}

            {% assign live_demo = project.demo_url | default: project.demo_link %}
            {% if live_demo %}
              <a href="{{ live_demo }}" target="_blank" rel="noopener noreferrer" class="project-link-icon" title="View Live Demo">
                <svg viewBox="0 0 16 16" width="20" height="20" fill="currentColor">
                  <path d="M3.75 2h3.5a.75.75 0 0 1 0 1.5h-3.5a.25.25 0 0 0-.25.25v8.5c0 .138.112.25.25.25h8.5a.25.25 0 0 0 .25-.25v-3.5a.75.75 0 0 1 1.5 0v3.5A1.75 1.75 0 0 1 12.25 14h-8.5A1.75 1.75 0 0 1 2 12.25v-8.5C2 2.784 2.784 2 3.75 2Zm6.854-1h4.146c.414 0 .75.336.75.75v4.146a.75.75 0 0 1-1.5 0V3.56L8.03 9.53a.75.75 0 0 1-1.06-1.06l5.97-5.97H11.354a.75.75 0 0 1 0-1.5Z"></path>
                </svg>
              </a>
            {% endif %}
          </div>
        </div>

        {% if project.description %}
          <p class="project-description">{{ project.description }}</p>
        {% endif %}

        {% if project.tags %}
          <div class="topic-tags" style="margin-top: 10px;">
            {% for tag in project.tags %}
              <a href="{{ '/projects/' | relative_url }}#{{ tag | slugify }}" class="tag-badge-link">
                  <span class="tag-badge">{{ tag }}</span>
              </a>
            {% endfor %}
          </div>
        {% endif %}
      </article>
      <hr class="project-divider">
    {% endif %}
  {% endfor %}
</div>

<!-- 4. Client-side JS Filter Script -->
<script>
  document.addEventListener("DOMContentLoaded", function () {
    const filterButtons = document.querySelectorAll(".filter-btn");
    const projectItems = document.querySelectorAll(".project-item");

    function applyFilter(targetTag) {
      // Toggle active button state
      filterButtons.forEach((btn) => {
        if (btn.getAttribute("data-tag") === targetTag) {
          btn.classList.add("active");
        } else {
          btn.classList.remove("active");
        }
      });

      // Filter project cards
      projectItems.forEach((item) => {
        const itemTags = item.getAttribute("data-tags") ? item.getAttribute("data-tags").split(" ") : [];
        const divider = item.nextElementSibling;

        if (targetTag === "all" || itemTags.includes(targetTag)) {
          item.style.display = "block";
          if (divider && divider.classList.contains("project-divider")) {
            divider.style.display = "block";
          }
        } else {
          item.style.display = "none";
          if (divider && divider.classList.contains("project-divider")) {
            divider.style.display = "none";
          }
        }
      });
    }

    // 1. Read hash from URL on page load (e.g., #python)
    function handleHashFilter() {
      const hash = window.location.hash.replace("#", "").toLowerCase();
      if (hash) {
        applyFilter(hash);
      } else {
        applyFilter("all");
      }
    }

    // Initialize filter from URL
    handleHashFilter();

    // Listen for hash changes if navigating on the same page
    window.addEventListener("hashchange", handleHashFilter);

    // 2. Click handler for filter buttons
    filterButtons.forEach((button) => {
      button.addEventListener("click", function () {
        const selectedTag = this.getAttribute("data-tag");
        if (selectedTag === "all") {
          history.pushState("", document.title, window.location.pathname);
          applyFilter("all");
        } else {
          window.location.hash = selectedTag;
        }
      });
    });
  });
</script>