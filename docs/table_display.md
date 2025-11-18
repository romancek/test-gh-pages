---
layout: default
title: Table Display
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
  <input type="text" id="filter" placeholder="Enter value to filter...">
</p>

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
    {% for i in (0..site.data.table_data.size | minus: 1) %}
      {% assign current_row = site.data.table_data[i] %}
      {% assign next_row = site.data.table_data[i | plus: 1] %}
      
      {% assign current_note = current_row.note | default: "" | strip %}
      {% assign next_note = next_row.note | default: "" | strip %}
      
      <!-- セル結合判定: 現在のnoteが非空かつ次のnoteが空なら結合 -->
      {% assign should_merge = false %}
      {% if current_note != "" and next_note == "" %}
        {% assign should_merge = true %}
      {% endif %}
      
      <tr class="data-row" data-item3="{{ current_row.item3 }}">
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
        <tr class="data-row" data-item3="{{ next_row.item3 }}">
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
    const filterValue = e.target.value.toLowerCase().trim();
    const rows = document.querySelectorAll('#data-table tbody tr.data-row');
    
    rows.forEach(row => {
      const item3Value = row.getAttribute('data-item3').toLowerCase();
      
      if (filterValue === '' || item3Value.includes(filterValue)) {
        row.style.display = '';
      } else {
        row.style.display = 'none';
      }
    });
  });
</script>
