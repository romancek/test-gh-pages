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
  <label for="filter-column-1">Column 1:</label>
  <select id="filter-column-1" style="padding: 5px; font-size: 14px; margin-right: 5px;">
    <option value="">-- None --</option>
    <option value="0">item1</option>
    <option value="1">item2</option>
    <option value="2">item3</option>
  </select>
  <input type="text" id="filter-value-1" placeholder="Value..." style="padding: 5px; font-size: 14px; margin-right: 20px;">
  
  <label for="filter-column-2">Column 2:</label>
  <select id="filter-column-2" style="padding: 5px; font-size: 14px; margin-right: 5px;">
    <option value="">-- None --</option>
    <option value="0">item1</option>
    <option value="1">item2</option>
    <option value="2">item3</option>
  </select>
  <input type="text" id="filter-value-2" placeholder="Value..." style="padding: 5px; font-size: 14px;">
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
      
      // フィルタ要素を取得
      this.filterColumn1 = document.getElementById('filter-column-1');
      this.filterValue1 = document.getElementById('filter-value-1');
      this.filterColumn2 = document.getElementById('filter-column-2');
      this.filterValue2 = document.getElementById('filter-value-2');
      
      // イベントリスナーを登録
      this.filterColumn1.addEventListener('change', () => this.applyFilter());
      this.filterValue1.addEventListener('keyup', () => this.applyFilter());
      this.filterColumn2.addEventListener('change', () => this.applyFilter());
      this.filterValue2.addEventListener('keyup', () => this.applyFilter());
      
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
      const filters = [
        {
          column: this.filterColumn1.value,
          value: this.filterValue1.value.toLowerCase().trim()
        },
        {
          column: this.filterColumn2.value,
          value: this.filterValue2.value.toLowerCase().trim()
        }
      ];
      
      this.rows.forEach(row => {
        let isMatch = true;
        
        // すべてのフィルタ条件をチェック（AND条件）
        for (const filter of filters) {
          // フィルタが設定されていない場合はスキップ
          if (!filter.column || !filter.value) {
            continue;
          }
          
          const columnIndex = parseInt(filter.column);
          const cell = row.querySelector(`td:nth-child(${columnIndex + 1})`);
          const cellValue = cell ? cell.textContent.toLowerCase() : '';
          
          // このフィルタ条件に一致しない場合、行全体が不一致
          if (!cellValue.includes(filter.value)) {
            isMatch = false;
            break;
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
