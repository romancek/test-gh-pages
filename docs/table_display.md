---
layout: default
title: Table Display
filter_item3: foo
---

<style>
  table {
    border-collapse: collapse;
    width: 100%;
  }
  th, td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }
  th {
    background-color: #f2f2f2;
  }
  .note-cell-merged {
    vertical-align: top;
  }
</style>

<h1>Table Data</h1>

<p>
  <label for="filter">Filter by item3:</label>
  <input type="text" id="filter" placeholder="Enter value to filter..." value="{{ page.filter_item3 }}">
</p>

{% assign filtered_data = "" | split: "" %}
{% for row in site.data.table_data %}
  {% if row.item3 == page.filter_item3 or page.filter_item3 == "" %}
    {% assign filtered_data = filtered_data | push: row %}
  {% endif %}
{% endfor %}

<table id="data-table">
  <thead>
    <tr>
      <th>item1</th>
      <th>item2</th>
      <th>item3</th>
      <th>note</th>
    </tr>
  </thead>
  <tbody>
    {% for i in (0..filtered_data.size | minus: 1) %}
      {% assign current_row = filtered_data[i] %}
      {% assign next_row = filtered_data[i | plus: 1] %}
      
      {% assign current_note = current_row.note | default: "" %}
      {% assign next_note = next_row.note | default: "" %}
      
      <!-- セル結合判定: 現在のnoteが非空かつ次のnoteが空なら結合 -->
      {% assign should_merge = false %}
      {% if current_note != "" and next_note == "" %}
        {% assign should_merge = true %}
      {% endif %}
      
      <tr>
        <td>{{ current_row.item1 }}</td>
        <td>{{ current_row.item2 }}</td>
        <td>{{ current_row.item3 }}</td>
        <td class="{% if should_merge %}note-cell-merged{% endif %}" 
            {% if should_merge %}rowspan="2"{% endif %}>
          {{ current_note }}
        </td>
      </tr>
      
      <!-- 次の行がnoteが空で現在がnoteが非空の場合、次の行はnoteセルを出力しない -->
      {% if should_merge %}
        <tr>
          <td>{{ next_row.item1 }}</td>
          <td>{{ next_row.item2 }}</td>
          <td>{{ next_row.item3 }}</td>
        </tr>
        {% assign i = i | plus: 1 %}
      {% endif %}
    {% endfor %}
  </tbody>
</table>

<script>
  document.getElementById('filter').addEventListener('keyup', function(e) {
    const filterValue = e.target.value.toLowerCase();
    const rows = document.querySelectorAll('#data-table tbody tr');
    
    rows.forEach(row => {
      const item3Cell = row.cells[2].textContent.toLowerCase();
      if (filterValue === '' || item3Cell.includes(filterValue)) {
        row.style.display = '';
      } else {
        row.style.display = 'none';
      }
    });
  });
</script>
