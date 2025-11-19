---
title: Tree Display
layout: tree
---
<h1>Tree Structure Data (col1 & col2)</h1>
<div id="tree-container"></div>
<script>
  class TreeRenderer {
    constructor(containerId, data) {
      this.container = document.getElementById(containerId);
      this.rawData = data;
      this.treeData = [];
      this.expandedNodes = new Set();
      this.buildTree();
      this.render();
      this.attachEventListeners();
    }
    buildTree() {
      const col1Groups = new Map();
      this.rawData.forEach(item => {
        const col1 = item.col1;
        const col2 = item.col2 || '';
        if (!col1Groups.has(col1)) {
          col1Groups.set(col1, new Map());
        }
        const col2Map = col1Groups.get(col1);
        if (!col2Map.has(col2)) {
          col2Map.set(col2, []);
        }
        col2Map.get(col2).push(item);
      });
      let level1Index = 0;
      col1Groups.forEach((col2Map, col1Value) => {
        const col1Node = {
          id: `col1_${level1Index}`,
          name: col1Value,
          level: 1,
          children: []
        };
        let level2Index = 0;
        col2Map.forEach((items, col2Value) => {
          if (col2Value === '') {
            col1Node.items = items;
            col1Node.hasDirectItems = true;
          } else {
            const col2Node = {
              id: `col1_${level1Index}_col2_${level2Index}`,
              name: col2Value,
              level: 2,
              itemCount: items.length,
              items: items
            };
            col1Node.children.push(col2Node);
            level2Index++;
          }
        });
        this.treeData.push(col1Node);
        level1Index++;
      });
    }
    render() {
      this.container.innerHTML = '';
      this.treeData.forEach(node => {
        this.container.appendChild(this.renderNode(node));
      });
    }
    renderNode(node) {
      const nodeEl = document.createElement('div');
      nodeEl.className = 'tree-node';
      nodeEl.dataset.id = node.id;
      const contentEl = document.createElement('div');
      contentEl.className = `tree-node-content level${node.level}`;
      const hasChildren = (node.children && node.children.length > 0) || node.hasDirectItems;
      const toggleEl = document.createElement('span');
      toggleEl.className = 'tree-toggle';
      if (hasChildren) {
        toggleEl.classList.add('collapsed');
      } else {
        toggleEl.classList.add('leaf');
      }
      contentEl.appendChild(toggleEl);
      const nameEl = document.createElement('span');
      nameEl.className = 'tree-node-name';
      nameEl.textContent = node.name;
      contentEl.appendChild(nameEl);
      if (node.level === 1 && node.hasDirectItems) {
        const countEl = document.createElement('span');
        countEl.className = 'tree-item-count';
        countEl.textContent = `(${node.items.length} items)`;
        contentEl.appendChild(countEl);
      } else if (node.level === 2 && node.itemCount) {
        const countEl = document.createElement('span');
        countEl.className = 'tree-item-count';
        countEl.textContent = `(${node.itemCount} items)`;
        contentEl.appendChild(countEl);
      }
      const childrenEl = document.createElement('div');
      childrenEl.className = 'tree-children';
      contentEl.addEventListener('click', (e) => {
        e.stopPropagation();
        this.toggleNode(node.id, toggleEl, childrenEl);
      });
      nodeEl.appendChild(contentEl);
      if (hasChildren) {
        if (node.hasDirectItems) {
          childrenEl.appendChild(this.renderItemsTable(node.items));
        }
        if (node.children && node.children.length > 0) {
          node.children.forEach(child => {
            childrenEl.appendChild(this.renderLevel2Node(child));
          });
        }
      }
      nodeEl.appendChild(childrenEl);
      return nodeEl;
    }
    renderLevel2Node(node) {
      const nodeEl = document.createElement('div');
      nodeEl.className = 'tree-node';
      nodeEl.dataset.id = node.id;
      const contentEl = document.createElement('div');
      contentEl.className = 'tree-node-content level2';
      const toggleEl = document.createElement('span');
      toggleEl.className = 'tree-toggle leaf';
      contentEl.appendChild(toggleEl);
      const nameEl = document.createElement('span');
      nameEl.className = 'tree-node-name';
      nameEl.textContent = node.name;
      contentEl.appendChild(nameEl);
      const countEl = document.createElement('span');
      countEl.className = 'tree-item-count';
      countEl.textContent = `(${node.itemCount} items)`;
      contentEl.appendChild(countEl);
      nodeEl.appendChild(contentEl);
      const tableEl = this.renderItemsTable(node.items);
      nodeEl.appendChild(tableEl);
      return nodeEl;
    }
    renderItemsTable(items) {
      const tableEl = document.createElement('div');
      tableEl.className = 'tree-item-table-wrapper';
      const table = document.createElement('table');
      const thead = document.createElement('thead');
      const headerRow = document.createElement('tr');
      ['col3', 'col4', 'col5', 'col6', 'col7', 'col8', 'col9', 'col10'].forEach(col => {
        const th = document.createElement('th');
        th.textContent = col;
        headerRow.appendChild(th);
      });
      thead.appendChild(headerRow);
      table.appendChild(thead);
      const tbody = document.createElement('tbody');
      items.forEach((item, idx) => {
        const row = document.createElement('tr');
        ['col3', 'col4', 'col5', 'col6', 'col7', 'col8', 'col9', 'col10'].forEach(col => {
          const td = document.createElement('td');
          td.textContent = item[col];
          row.appendChild(td);
        });
        tbody.appendChild(row);
      });
      table.appendChild(tbody);
      tableEl.appendChild(table);
      return tableEl;
    }
    toggleNode(nodeId, toggleEl, childrenEl) {
      if (this.expandedNodes.has(nodeId)) {
        this.expandedNodes.delete(nodeId);
        toggleEl.classList.remove('expanded');
        toggleEl.classList.add('collapsed');
        childrenEl.classList.remove('expanded');
      } else {
        this.expandedNodes.add(nodeId);
        toggleEl.classList.remove('collapsed');
        toggleEl.classList.add('expanded');
        childrenEl.classList.add('expanded');
      }
    }
    attachEventListeners() {
    }
  }
  document.addEventListener('DOMContentLoaded', function() {
    const treeData = {{ site.data.tree_data | jsonify }};
    if (treeData && treeData.length > 0) {
      new TreeRenderer('tree-container', treeData);
    }
  });
</script>
