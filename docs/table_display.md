---
layout: default
title: Table Display
---

<style>
  table {
    border-collapse: collapse;
    width: 100%;
    margin-top: 20px;
  }
  th, td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }
  th {
    background-color: #f2f2f2;
    font-weight: bold;
  }
  tr.hidden {
    display: none !important;
  }
</style>

<h1>Table Data</h1>

<p>
  <label for="filter">Filter by item3:</label>
  <input type="text" id="filter" placeholder="Enter value to filter..." style="padding: 5px; font-size: 14px;">
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
  <tbody id="table-body">
    {% for row in site.data.table_data %}
      <tr class="data-row" data-item3="{{ row.item3 }}" data-index="{{ forloop.index0 }}">
        <td>{{ row.item1 }}</td>
        <td>{{ row.item2 }}</td>
        <td>{{ row.item3 }}</td>
        <td class="note-cell">{{ row.note }}</td>
      </tr>
    {% endfor %}
  </tbody>
</table>

<script>
  // セル結合と行の非表示を管理するクラス
  class TableManager {
    constructor(tableId) {
      this.table = document.getElementById(tableId);
      this.tbody = document.getElementById('table-body');
      this.rows = Array.from(this.tbody.querySelectorAll('tr.data-row'));
      this.filterInput = document.getElementById('filter');
      
      this.filterInput.addEventListener('keyup', () => this.applyFilter());
      this.applyMerging();
    }
    
    // セル結合ロジック
    applyMerging() {
      const noteCells = this.tbody.querySelectorAll('td.note-cell');
      
      noteCells.forEach((cell, index) => {
        const currentNote = cell.textContent.trim();
        const nextCell = noteCells[index + 1];
        const nextNote = nextCell ? nextCell.textContent.trim() : '';
        
        // 現在のnoteが非空かつ次のnoteが空ならrowspan=2
        if (currentNote !== '' && nextNote === '') {
          cell.setAttribute('rowspan', '2');
          cell.style.verticalAlign = 'middle';
          // 次の行のnoteセルを削除
          if (nextCell) {
            nextCell.remove();
          }
        }
      });
    }
    
    // フィルタリング
    applyFilter() {
      const filterValue = this.filterInput.value.toLowerCase().trim();
      
      this.rows.forEach(row => {
        const item3Value = row.getAttribute('data-item3').toLowerCase();
        const isMatch = filterValue === '' || item3Value.includes(filterValue);
        
        if (isMatch) {
          row.classList.remove('hidden');
        } else {
          row.classList.add('hidden');
        }
      });
      
      // フィルタ後にセル結合を再適用
      this.reapplyMerging();
    }
    
    // フィルタ後のセル結合再適用
    reapplyMerging() {
      const visibleNoteCells = Array.from(this.tbody.querySelectorAll('td.note-cell:not([rowspan])')).filter(cell => {
        return cell.parentElement.classList.contains('data-row') && !cell.parentElement.classList.contains('hidden');
      });
      
      visibleNoteCells.forEach((cell, index) => {
        const currentNote = cell.textContent.trim();
        const nextCell = visibleNoteCells[index + 1];
        const nextNote = nextCell ? nextCell.textContent.trim() : '';
        
        if (currentNote !== '' && nextNote === '') {
          cell.setAttribute('rowspan', '2');
          cell.style.verticalAlign = 'middle';
        }
      });
    }
  }
  
  // ページ読み込み時に初期化
  document.addEventListener('DOMContentLoaded', function() {
    new TableManager('data-table');
  });
</script>
