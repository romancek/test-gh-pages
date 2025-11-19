---
title: Tree Display
layout: table
---

<h1>Tree Structure Data (col1 & col2)</h1>

<div id="tree-container"></div>

<style>
  /* Override layout styles */
  html, body {
    height: auto !important;
  }
  
  body {
    display: block !important;
  }

  .table-wrapper {
    flex: initial !important;
    overflow: visible !important;
    height: auto !important;
    min-height: auto !important;
    width: auto !important;
  }

  .table-container {
    overflow: visible !important;
    width: auto !important;
    height: auto !important;
  }

  #tree-container {
    padding: 20px;
    background-color: white;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  }

  .tree-node {
    margin-left: 0;
    margin-bottom: 0;
  }

  .tree-node-content {
    display: flex;
    align-items: center;
    padding: 10px;
    cursor: pointer;
    user-select: none;
    border-radius: 4px;
    transition: background-color 0.2s;
  }

  .tree-node-content:hover {
    background-color: #f0f0f0;
  }

  .tree-node-content.level1 {
    font-weight: bold;
    font-size: 16px;
    color: #1a73e8;
    padding-left: 0;
  }

  .tree-node-content.level2 {
    font-size: 14px;
    color: #34a853;
    padding-left: 32px;
  }

  .tree-toggle {
    display: inline-block;
    width: 20px;
    height: 20px;
    margin-right: 8px;
    text-align: center;
    line-height: 20px;
    font-size: 14px;
    font-weight: bold;
    color: #666;
  }

  .tree-toggle.collapsed::before {
    content: "▶";
  }

  .tree-toggle.expanded::before {
    content: "▼";
  }

  .tree-toggle.leaf {
    visibility: hidden;
  }

  .tree-children {
    display: none;
  }

  .tree-children.expanded {
    display: block;
  }

  .tree-node-name {
    flex: 1;
  }

  .tree-item-count {
    font-size: 12px;
    color: #999;
    margin-left: auto;
    padding-left: 20px;
    text-align: right;
  }
</style>

<script>
  class TreeRenderer {
    constructor(containerId, data) {
      this.container = document.getElementById(containerId);
      this.rawData = data;
      this.treeData = [];
      this.expandedNodes = new Set();
      
      // Build tree structure from flat data
      this.buildTree();
      this.render();
      this.attachEventListeners();
    }

    buildTree() {
      const col1Groups = new Map();
      
      // Group data by col1 and col2
      this.rawData.forEach(item => {
        const col1 = item.col1;
        const col2 = item.col2;
        
        if (!col1Groups.has(col1)) {
          col1Groups.set(col1, new Map());
        }
        
        const col2Map = col1Groups.get(col1);
        if (!col2Map.has(col2)) {
          col2Map.set(col2, []);
        }
        
        col2Map.get(col2).push(item);
      });
      
      // Convert to tree structure
      let level1Index = 0;
      col1Groups.forEach((col2Map, col1Value) => {
        const col1Node = {
          id: `col1_${col1Index}`,
          name: col1Value,
          level: 1,
          children: []
        };
        
        let level2Index = 0;
        col2Map.forEach((items, col2Value) => {
          const col2Node = {
            id: `col1_${col1Index}_col2_${level2Index}`,
            name: col2Value,
            level: 2,
            itemCount: items.length,
            items: items
          };
          col1Node.children.push(col2Node);
          level2Index++;
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

      const hasChildren = node.children && node.children.length > 0;

      // Toggle button
      const toggleEl = document.createElement('span');
      toggleEl.className = 'tree-toggle';
      if (hasChildren) {
        toggleEl.classList.add('collapsed');
      } else {
        toggleEl.classList.add('leaf');
      }
      contentEl.appendChild(toggleEl);

      // Node name
      const nameEl = document.createElement('span');
      nameEl.className = 'tree-node-name';
      nameEl.textContent = node.name;
      contentEl.appendChild(nameEl);

      // Item count for level2
      if (node.level === 2 && node.itemCount) {
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

      // Children container
      if (hasChildren) {
        node.children.forEach(child => {
          if (child.level === 2) {
            // For level2, render as a table
            childrenEl.appendChild(this.renderLevel2Node(child));
          } else {
            childrenEl.appendChild(this.renderNode(child));
          }
        });
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

      // No toggle for leaf level2
      const toggleEl = document.createElement('span');
      toggleEl.className = 'tree-toggle leaf';
      contentEl.appendChild(toggleEl);

      // Node name
      const nameEl = document.createElement('span');
      nameEl.className = 'tree-node-name';
      nameEl.textContent = node.name;
      contentEl.appendChild(nameEl);

      // Item count
      const countEl = document.createElement('span');
      countEl.className = 'tree-item-count';
      countEl.textContent = `(${node.itemCount} items)`;
      contentEl.appendChild(countEl);

      nodeEl.appendChild(contentEl);

      // Items table
      const tableEl = document.createElement('div');
      tableEl.style.paddingLeft = '32px';
      tableEl.style.paddingTop = '8px';
      tableEl.style.overflow = 'auto';

      const table = document.createElement('table');
      table.style.width = '100%';
      table.style.fontSize = '13px';
      table.style.borderCollapse = 'collapse';

      // Table header
      const thead = document.createElement('thead');
      const headerRow = document.createElement('tr');
      headerRow.style.backgroundColor = '#f8f9fa';
      
      ['col3', 'col4', 'col5', 'col6', 'col7', 'col8', 'col9', 'col10'].forEach(col => {
        const th = document.createElement('th');
        th.textContent = col;
        th.style.border = '1px solid #ddd';
        th.style.padding = '8px';
        th.style.fontWeight = 'bold';
        th.style.textAlign = 'left';
        headerRow.appendChild(th);
      });
      thead.appendChild(headerRow);
      table.appendChild(thead);

      // Table body
      const tbody = document.createElement('tbody');
      node.items.forEach((item, idx) => {
        const row = document.createElement('tr');
        if (idx % 2 === 1) {
          row.style.backgroundColor = '#f9f9f9';
        }
        
        ['col3', 'col4', 'col5', 'col6', 'col7', 'col8', 'col9', 'col10'].forEach(col => {
          const td = document.createElement('td');
          td.textContent = item[col];
          td.style.border = '1px solid #ddd';
          td.style.padding = '8px';
          row.appendChild(td);
        });
        tbody.appendChild(row);
      });
      table.appendChild(tbody);

      tableEl.appendChild(table);
      nodeEl.appendChild(tableEl);

      return nodeEl;
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
      // Default: keep all nodes collapsed (no expansion)
      // This is the default behavior - users can click to expand
    }
  }

  // Initialize tree on page load
  document.addEventListener('DOMContentLoaded', function() {
    const treeData = {{ site.data.tree_data | jsonify }};
    new TreeRenderer('tree-container', treeData);
  });
</script>
