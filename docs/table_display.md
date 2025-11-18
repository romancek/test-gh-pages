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
    {% assign skip_next = false %}
    {% for row in site.data.table_data %}
      {% if skip_next %}
        {% assign skip_next = false %}
        {% continue %}
      {% endif %}
      
      {% assign current_note = row.note | default: "" | strip %}
      {% assign row_index = forloop.index0 %}
      {% assign next_row = site.data.table_data[row_index | plus: 1] %}
      {% assign next_note = next_row.note | default: "" | strip %}
      
      {% assign should_merge = false %}
      {% if current_note != "" and next_note == "" %}
        {% assign should_merge = true %}
      {% endif %}
      
      <tr class="data-row" data-item3="{{ row.item3 }}">
        <td>{{ row.item1 }}</td>
        <td>{{ row.item2 }}</td>
        <td>{{ row.item3 }}</td>
        {% if should_merge %}
          <td rowspan="2">{{ current_note }}</td>
        {% else %}
          <td>{{ current_note }}</td>
        {% endif %}
      </tr>
      
      {% if should_merge %}
        <tr class="data-row" data-item3="{{ next_row.item3 }}">
          <td>{{ next_row.item1 }}</td>
          <td>{{ next_row.item2 }}</td>
          <td>{{ next_row.item3 }}</td>
        </tr>
        {% assign skip_next = true %}
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
