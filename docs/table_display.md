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
  <label for="filter-column">Filter column:</label>
  <select id="filter-column" style="padding: 5px; font-size: 14px; margin-right: 10px;">
    <option value="all">All columns</option>
    <option value="0">item1</option>
    <option value="1">item2</option>
    <option value="2">item3</option>
  </select>
  
  <label for="filter">Filter value:</label>
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
      this.filterColumn = document.getElementById('filter-column');
      
      this.filterInput.addEventListener('keyup', () => this.applyFilter());
      this.filterColumn.addEventListener('change', () => this.applyFilter());
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
      const filterColumnValue = this.filterColumn.value;
      
      this.rows.forEach(row => {
        let isMatch = filterValue === '';
        
        if (filterValue !== '') {
          if (filterColumnValue === 'all') {
            // すべてのカラム（note以外）をチェック
            const cells = Array.from(row.querySelectorAll('td')).slice(0, -1); // note セルは除外
            isMatch = cells.some(cell => cell.textContent.toLowerCase().includes(filterValue));
          } else {
            // 指定されたカラムのみチェック
            const columnIndex = parseInt(filterColumnValue);
            const cell = row.querySelector(`td:nth-child(${columnIndex + 1})`);
            isMatch = cell ? cell.textContent.toLowerCase().includes(filterValue) : false;
          }
        }
        
        row.classList.toggle('hidden', !isMatch);
      });
      
      // フィルタ後にセル結合を再適用
      this.reapplyMerging();
    }
    
    // フィルタ後のセル結合再適用
    reapplyMerging() {
      // 既存の rowspan をリセット
      const allNoteCells = this.tbody.querySelectorAll('td.note-cell');
      allNoteCells.forEach(cell => {
        cell.removeAttribute('rowspan');
        cell.style.verticalAlign = 'top';
      });
      
      // 表示中の行のnoteセルのみを取得
      const visibleRows = Array.from(this.rows).filter(row => !row.classList.contains('hidden'));
      const visibleNoteCells = visibleRows.map(row => row.querySelector('td.note-cell'));
      
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
