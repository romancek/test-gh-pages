---
title: Table Display
layout: table
---

<h1>Table Data</h1>

<p>
  <label for="filter-column-1">Column 1:</label>
  <select id="filter-column-1" style="padding: 5px; font-size: 14px; margin-right: 5px;">
    <option value="">-- None --</option>
    <option value="0">col1</option>
    <option value="1">col2</option>
    <option value="2">col3</option>
    <option value="3">col4</option>
    <option value="4">col5</option>
    <option value="5">col6</option>
    <option value="6">col7</option>
    <option value="7">col8</option>
    <option value="8">col9</option>
    <option value="9">col10</option>
  </select>
  <input type="text" id="filter-value-1" placeholder="Value..." style="padding: 5px; font-size: 14px; margin-right: 20px;">
  
  <label for="filter-column-2">Column 2:</label>
  <select id="filter-column-2" style="padding: 5px; font-size: 14px; margin-right: 5px;">
    <option value="">-- None --</option>
    <option value="0">col1</option>
    <option value="1">col2</option>
    <option value="2">col3</option>
    <option value="3">col4</option>
    <option value="4">col5</option>
    <option value="5">col6</option>
    <option value="6">col7</option>
    <option value="7">col8</option>
    <option value="8">col9</option>
    <option value="9">col10</option>
  </select>
  <input type="text" id="filter-value-2" placeholder="Value..." style="padding: 5px; font-size: 14px;">
</p>

<div class="scroll-container-top" id="scroll-top">
  <div class="scroll-spacer"></div>
</div>

<div class="table-wrapper">
  <div class="table-container" id="scroll-bottom">
    <table id="data-table">
  <thead>
    <tr>
      <th>col1</th>
      <th>col2</th>
      <th>col3</th>
      <th>col4</th>
      <th>col5</th>
      <th>col6</th>
      <th>col7</th>
      <th>col8</th>
      <th>col9</th>
      <th>col10</th>
    </tr>
  </thead>
  <tbody id="table-body">
    {% for row in site.data.table_data %}
      <tr class="data-row" data-index="{{ forloop.index0 }}">
        <td>{{ row.col1 }}</td>
        <td>{{ row.col2 }}</td>
        <td>{{ row.col3 }}</td>
        <td>{{ row.col4 }}</td>
        <td>{{ row.col5 }}</td>
        <td>{{ row.col6 }}</td>
        <td>{{ row.col7 }}</td>
        <td>{{ row.col8 }}</td>
        <td>{{ row.col9 }}</td>
        <td>{{ row.col10 }}</td>
      </tr>
    {% endfor %}
  </tbody>
</table>
  </div>
</div>

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
      
      // スクロール要素を取得
      this.scrollTop = document.getElementById('scroll-top');
      this.scrollBottom = document.getElementById('scroll-bottom');
      this.scrollSpacer = this.scrollTop.querySelector('.scroll-spacer');
      
      // イベントリスナーを登録
      this.filterColumn1.addEventListener('change', () => this.applyFilter());
      this.filterValue1.addEventListener('keyup', () => this.applyFilter());
      this.filterColumn2.addEventListener('change', () => this.applyFilter());
      this.filterValue2.addEventListener('keyup', () => this.applyFilter());
      
      // スクロール同期
      this.scrollTop.addEventListener('scroll', () => {
        this.scrollBottom.scrollLeft = this.scrollTop.scrollLeft;
      });
      this.scrollBottom.addEventListener('scroll', () => {
        this.scrollTop.scrollLeft = this.scrollBottom.scrollLeft;
      });
      
      // 上部スクロールバーの幅を設定
      setTimeout(() => this.setSyncScrollWidth(), 100);
      window.addEventListener('resize', () => this.setSyncScrollWidth());
      
      this.applyMerging();
    }
    
    // 上部スクロール領域の幅を動的に設定
    setSyncScrollWidth() {
      const scrollWidth = this.scrollBottom.scrollWidth;
      const clientWidth = this.scrollBottom.clientWidth;
      
      if (scrollWidth > clientWidth) {
        this.scrollSpacer.style.width = scrollWidth + 'px';
      }
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
