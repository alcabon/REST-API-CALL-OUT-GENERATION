# **BEST DATATABLE LIBRARIES FOR VSCODE EXTENSION - YAML SCHEMA GENERATOR**

## **🏆 RECOMMENDED: AG-GRID** (Enterprise-Grade)

### **Why AG-Grid is the Best Choice:**

```typescript
// ag-grid-community (free, open source)
// Perfect for VSCode extension context

import { AgGridReact } from 'ag-grid-react';
import 'ag-grid-community/styles/ag-grid.css';
import 'ag-grid-community/styles/ag-theme-vscode-dark.css';

const columnDefs = [
  {
    field: 'objectName',
    headerName: 'Object/Field',
    cellRenderer: 'agGroupCellRenderer',
    checkboxSelection: true,
    headerCheckboxSelection: true,
  },
  { field: 'type', headerName: 'Type' },
  { field: 'label', headerName: 'Label' },
  { field: 'required', headerName: 'Required', cellRenderer: 'booleanRenderer' },
  { field: 'length', headerName: 'Length' },
];

const rowData = [
  {
    objectName: 'Account',
    type: 'Standard Object',
    orgHierarchy: ['Account'],
    children: [
      {
        objectName: 'Name',
        type: 'Text',
        label: 'Account Name',
        required: true,
        length: 255,
        orgHierarchy: ['Account', 'Name'],
      },
      {
        objectName: 'YearStarted__c',
        type: 'Number',
        label: 'Year Started',
        required: false,
        length: 4,
        orgHierarchy: ['Account', 'YearStarted__c'],
      },
    ],
  },
  {
    objectName: 'Contact',
    type: 'Standard Object',
    orgHierarchy: ['Contact'],
    children: [
      {
        objectName: 'FirstName',
        type: 'Text',
        label: 'First Name',
        required: false,
        length: 40,
        orgHierarchy: ['Contact', 'FirstName'],
      },
    ],
  },
];

<AgGridReact
  columnDefs={columnDefs}
  rowData={rowData}
  treeData={true}
  animateRows={true}
  groupDefaultExpanded={-1}
  getDataPath={(data) => data.orgHierarchy}
  autoGroupColumnDef={{
    headerName: 'Salesforce Schema',
    minWidth: 300,
    cellRendererParams: {
      checkbox: true,
      suppressCount: true,
    },
  }}
  rowSelection="multiple"
  onSelectionChanged={onSelectionChanged}
/>
```

### **Advantages:**
✅ **Tree Data Native** - Built-in hierarchical support  
✅ **Performance** - Virtual scrolling handles 10,000+ rows  
✅ **VSCode Theme** - Pre-built dark/light themes  
✅ **Checkbox Selection** - Parent-child checkbox logic  
✅ **TypeScript** - Full type safety  
✅ **Bundle Size** - 200KB gzipped (community edition)  
✅ **Active Development** - Updated monthly  
✅ **Export Built-in** - CSV, Excel export  

---

## **🥈 ALTERNATIVE 1: TANSTACK TABLE (React Table v8)** - Lightweight

```typescript
import { useReactTable, getCoreRowModel, getExpandedRowModel } from '@tanstack/react-table';

// Perfect for smaller datasets, ultra-lightweight
const table = useReactTable({
  data,
  columns,
  getCoreRowModel: getCoreRowModel(),
  getExpandedRowModel: getExpandedRowModel(),
  getSubRows: (row) => row.children,
  enableRowSelection: true,
  enableMultiRowSelection: true,
});

// Fully headless - complete control over rendering
```

### **Advantages:**
✅ **Headless** - Complete UI control  
✅ **Tiny Bundle** - 14KB gzipped  
✅ **React 18** - Modern hooks API  
✅ **TypeScript First** - Excellent types  
✅ **Extensible** - Plugin architecture  

### **Disadvantages:**
❌ No built-in UI (must build yourself)  
❌ More setup required  
❌ No virtual scrolling out-of-box  

---

## **🥉 ALTERNATIVE 2: REACT-COMPLEX-TREE** - Purpose-Built for Trees

```typescript
import { Tree, UncontrolledTreeEnvironment } from 'react-complex-tree';
import 'react-complex-tree/lib/style-modern.css';

const tree = {
  root: {
    index: 'root',
    isFolder: true,
    children: ['account', 'contact'],
    data: 'Salesforce Objects',
  },
  account: {
    index: 'account',
    isFolder: true,
    children: ['account-name', 'account-website'],
    data: { name: 'Account', type: 'Standard Object' },
  },
  'account-name': {
    index: 'account-name',
    data: { name: 'Name', type: 'Text', length: 255, required: true },
  },
};

<UncontrolledTreeEnvironment
  dataProvider={dataProvider}
  getItemTitle={(item) => item.data.name}
  viewState={{}}
  canDragAndDrop={true}
  canDropOnFolder={true}
  canReorderItems={true}
>
  <Tree treeId="salesforce-schema" rootItem="root" />
</UncontrolledTreeEnvironment>
```

### **Advantages:**
✅ **Tree-Specific** - Built for hierarchical data  
✅ **Drag & Drop** - Reorder fields easily  
✅ **Keyboard Navigation** - Accessible  
✅ **Search** - Built-in filtering  
✅ **VSCode-Like** - Familiar UX  

### **Disadvantages:**
❌ Not a traditional "table"  
❌ Limited column customization  

---

## **COMPARISON MATRIX**

| Feature | AG-Grid | TanStack Table | React-Complex-Tree | Tabulator |
|---------|---------|----------------|-------------------|-----------|
| **Tree/Hierarchy** | ⭐⭐⭐⭐⭐ Native | ⭐⭐⭐⭐ Via getSubRows | ⭐⭐⭐⭐⭐ Native | ⭐⭐⭐ Via row groups |
| **Checkboxes** | ⭐⭐⭐⭐⭐ Built-in | ⭐⭐⭐⭐ Manual | ⭐⭐⭐⭐⭐ Built-in | ⭐⭐⭐⭐ Built-in |
| **Performance** | ⭐⭐⭐⭐⭐ 10K+ rows | ⭐⭐⭐⭐ Good | ⭐⭐⭐⭐ Good | ⭐⭐⭐⭐ 5K+ rows |
| **VSCode Theme** | ⭐⭐⭐⭐⭐ Built-in | ⭐⭐⭐ Custom CSS | ⭐⭐⭐⭐ Customizable | ⭐⭐⭐ Custom CSS |
| **TypeScript** | ⭐⭐⭐⭐⭐ Excellent | ⭐⭐⭐⭐⭐ Excellent | ⭐⭐⭐⭐ Good | ⭐⭐⭐ Types available |
| **Bundle Size** | 200KB gzip | 14KB gzip | 50KB gzip | 150KB gzip |
| **Setup Effort** | ⭐⭐⭐⭐ Easy | ⭐⭐ Complex | ⭐⭐⭐⭐ Easy | ⭐⭐⭐⭐ Easy |
| **Customization** | ⭐⭐⭐⭐⭐ Infinite | ⭐⭐⭐⭐⭐ Infinite | ⭐⭐⭐ Limited | ⭐⭐⭐⭐ Good |
| **VSCode Webview** | ⭐⭐⭐⭐⭐ Perfect | ⭐⭐⭐⭐⭐ Perfect | ⭐⭐⭐⭐⭐ Perfect | ⭐⭐⭐⭐ Works |
| **Documentation** | ⭐⭐⭐⭐⭐ Excellent | ⭐⭐⭐⭐ Good | ⭐⭐⭐⭐ Good | ⭐⭐⭐⭐ Good |
| **Active Dev** | ⭐⭐⭐⭐⭐ Very active | ⭐⭐⭐⭐⭐ Very active | ⭐⭐⭐⭐ Active | ⭐⭐⭐ Moderate |
| **Export Data** | ⭐⭐⭐⭐⭐ Built-in | ⭐⭐ Manual | ⭐⭐ Manual | ⭐⭐⭐⭐⭐ Built-in |

---

## **🎯 MY EXPERT RECOMMENDATION**

### **For Your Use Case (VSCode Extension - Schema Generator):**

**Use AG-Grid Community** because:

1. ✅ **Perfect for VSCode Webviews** - Has official VSCode theme
2. ✅ **Tree + Table Hybrid** - Exactly what you need
3. ✅ **Checkbox Selection Logic** - Parent/child selection handled
4. ✅ **Performance** - Handles entire Salesforce schema (100+ objects)
5. ✅ **Zero Config Tree** - Just set `treeData={true}`
6. ✅ **Free & Open Source** - Community edition has everything you need
7. ✅ **Export to CSV/Excel** - Bonus feature for users
8. ✅ **Filter/Search Built-in** - Find fields quickly
9. ✅ **Keyboard Navigation** - Power user friendly
10. ✅ **TypeScript Native** - Type-safe development

---

## **COMPLETE VSCODE EXTENSION EXAMPLE**

### **Architecture:**

```
salesforce-yaml-generator/
├── src/
│   ├── extension.ts              # VSCode extension entry
│   ├── webview/
│   │   ├── SchemaTreeView.tsx    # Main React component
│   │   ├── YamlGenerator.ts      # YAML generation logic
│   │   ├── SalesforceApi.ts      # Fetch schema from org
│   │   └── index.tsx             # Webview entry
│   └── panels/
│       └── SchemaPanel.ts        # Webview panel manager
├── package.json
└── webpack.config.js
```

### **1. Extension Entry Point** (`extension.ts`)

```typescript
import * as vscode from 'vscode';
import { SchemaPanel } from './panels/SchemaPanel';

export function activate(context: vscode.ExtensionContext) {
  // Register command to open schema generator
  const openCommand = vscode.commands.registerCommand(
    'salesforce-yaml-generator.openSchemaGenerator',
    () => {
      SchemaPanel.render(context.extensionUri);
    }
  );

  context.subscriptions.push(openCommand);
}
```

### **2. Webview Panel Manager** (`SchemaPanel.ts`)

```typescript
import * as vscode from 'vscode';
import { getNonce } from '../utilities/getNonce';

export class SchemaPanel {
  public static currentPanel: SchemaPanel | undefined;
  private readonly _panel: vscode.WebviewPanel;
  private _disposables: vscode.Disposable[] = [];

  private constructor(panel: vscode.WebviewPanel, extensionUri: vscode.Uri) {
    this._panel = panel;
    this._panel.webview.html = this._getWebviewContent(
      this._panel.webview,
      extensionUri
    );
    
    // Handle messages from webview
    this._setWebviewMessageListener(this._panel.webview);
    
    // Clean up
    this._panel.onDidDispose(() => this.dispose(), null, this._disposables);
  }

  public static render(extensionUri: vscode.Uri) {
    if (SchemaPanel.currentPanel) {
      SchemaPanel.currentPanel._panel.reveal(vscode.ViewColumn.One);
    } else {
      const panel = vscode.window.createWebviewPanel(
        'salesforceSchemaGenerator',
        'Salesforce Schema Generator',
        vscode.ViewColumn.One,
        {
          enableScripts: true,
          retainContextWhenHidden: true,
          localResourceRoots: [
            vscode.Uri.joinPath(extensionUri, 'out'),
            vscode.Uri.joinPath(extensionUri, 'webview-ui/build'),
          ],
        }
      );

      SchemaPanel.currentPanel = new SchemaPanel(panel, extensionUri);
    }
  }

  private _setWebviewMessageListener(webview: vscode.Webview) {
    webview.onDidReceiveMessage(
      async (message: any) => {
        switch (message.command) {
          case 'generateYaml':
            await this._handleGenerateYaml(message.selectedItems);
            break;
          case 'fetchSchema':
            await this._handleFetchSchema();
            break;
          case 'saveYaml':
            await this._handleSaveYaml(message.yaml);
            break;
        }
      },
      undefined,
      this._disposables
    );
  }

  private async _handleGenerateYaml(selectedItems: any[]) {
    // Generate YAML from selected items
    const yaml = this._generateYamlContent(selectedItems);
    
    // Send back to webview
    this._panel.webview.postMessage({
      command: 'yamlGenerated',
      yaml: yaml,
    });
  }

  private async _handleFetchSchema() {
    try {
      // Fetch schema from Salesforce org
      const schema = await this._fetchSalesforceSchema();
      
      this._panel.webview.postMessage({
        command: 'schemaFetched',
        schema: schema,
      });
    } catch (error) {
      vscode.window.showErrorMessage(`Error fetching schema: ${error}`);
    }
  }

  private async _handleSaveYaml(yaml: string) {
    const uri = await vscode.window.showSaveDialog({
      defaultUri: vscode.Uri.file('salesforce-schema.yaml'),
      filters: { 'YAML Files': ['yaml', 'yml'] },
    });

    if (uri) {
      await vscode.workspace.fs.writeFile(uri, Buffer.from(yaml, 'utf8'));
      vscode.window.showInformationMessage('YAML file saved successfully!');
    }
  }

  private _generateYamlContent(selectedItems: any[]): string {
    // YAML generation logic
    return `# Salesforce Schema
version: "1.0"
objects:
${selectedItems.map(item => this._generateObjectYaml(item)).join('\n')}`;
  }

  private _generateObjectYaml(item: any): string {
    if (item.type === 'object') {
      return `  ${item.apiName}:
    type: ${item.objectType}
    label: "${item.label}"
    fields:
${item.fields?.map((f: any) => this._generateFieldYaml(f)).join('\n') || ''}`;
    }
    return '';
  }

  private _generateFieldYaml(field: any): string {
    return `      - apiName: ${field.apiName}
        label: "${field.label}"
        type: ${field.type}
        required: ${field.required}
        length: ${field.length || 'null'}`;
  }

  private async _fetchSalesforceSchema() {
    // Integration with Salesforce CLI or REST API
    const terminal = vscode.window.createTerminal('Salesforce');
    
    // Example: using Salesforce CLI
    return new Promise((resolve) => {
      terminal.sendText('sf sobject list --json');
      // Parse and return schema
      resolve({
        objects: [
          {
            apiName: 'Account',
            label: 'Account',
            type: 'standard',
            fields: [
              { apiName: 'Name', label: 'Account Name', type: 'string', required: true, length: 255 },
              { apiName: 'Industry', label: 'Industry', type: 'picklist', required: false },
            ],
          },
        ],
      });
    });
  }

  private _getWebviewContent(webview: vscode.Webview, extensionUri: vscode.Uri) {
    const scriptUri = webview.asWebviewUri(
      vscode.Uri.joinPath(extensionUri, 'out', 'webview.js')
    );
    const styleUri = webview.asWebviewUri(
      vscode.Uri.joinPath(extensionUri, 'out', 'webview.css')
    );

    const nonce = getNonce();

    return `<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="Content-Security-Policy" content="default-src 'none'; style-src ${webview.cspSource} 'unsafe-inline'; script-src 'nonce-${nonce}';">
  <link href="${styleUri}" rel="stylesheet">
  <title>Salesforce Schema Generator</title>
</head>
<body>
  <div id="root"></div>
  <script nonce="${nonce}" src="${scriptUri}"></script>
</body>
</html>`;
  }

  public dispose() {
    SchemaPanel.currentPanel = undefined;
    this._panel.dispose();
    while (this._disposables.length) {
      const disposable = this._disposables.pop();
      if (disposable) {
        disposable.dispose();
      }
    }
  }
}
```

### **3. React Component with AG-Grid** (`SchemaTreeView.tsx`)

```typescript
import React, { useState, useCallback, useMemo, useRef } from 'react';
import { AgGridReact } from 'ag-grid-react';
import { ColDef, IRowNode } from 'ag-grid-community';
import 'ag-grid-community/styles/ag-grid.css';
import 'ag-grid-community/styles/ag-theme-vscode-dark.css';

interface SchemaNode {
  id: string;
  objectName: string;
  apiName: string;
  type: string;
  label: string;
  required?: boolean;
  length?: number;
  dataType?: string;
  orgHierarchy: string[];
  children?: SchemaNode[];
}

export const SchemaTreeView: React.FC = () => {
  const gridRef = useRef<AgGridReact>(null);
  const [rowData, setRowData] = useState<SchemaNode[]>([]);
  const [selectedRows, setSelectedRows] = useState<SchemaNode[]>([]);

  // Column definitions
  const columnDefs: ColDef[] = useMemo(() => [
    {
      field: 'objectName',
      headerName: 'Object/Field Name',
      minWidth: 300,
      flex: 1,
    },
    {
      field: 'type',
      headerName: 'Type',
      width: 150,
      cellStyle: { fontFamily: 'monospace' },
    },
    {
      field: 'label',
      headerName: 'Label',
      width: 200,
    },
    {
      field: 'required',
      headerName: 'Required',
      width: 100,
      cellRenderer: (params: any) => {
        if (params.value === undefined) return '';
        return params.value ? '✓' : '✗';
      },
    },
    {
      field: 'length',
      headerName: 'Length',
      width: 100,
      cellRenderer: (params: any) => params.value || '-',
    },
    {
      field: 'dataType',
      headerName: 'Data Type',
      width: 120,
    },
  ], []);

  // Auto group column definition
  const autoGroupColumnDef: ColDef = useMemo(() => ({
    headerName: 'Salesforce Schema',
    minWidth: 300,
    cellRendererParams: {
      checkbox: true,
      suppressCount: true,
      innerRenderer: (params: any) => {
        return params.data.objectName;
      },
    },
  }), []);

  // Default column settings
  const defaultColDef: ColDef = useMemo(() => ({
    sortable: true,
    filter: true,
    resizable: true,
  }), []);

  // Fetch schema from extension
  const fetchSchema = useCallback(() => {
    // @ts-ignore
    if (typeof vscode !== 'undefined') {
      // @ts-ignore
      vscode.postMessage({ command: 'fetchSchema' });
    }
  }, []);

  // Handle selection change
  const onSelectionChanged = useCallback(() => {
    const selectedNodes = gridRef.current?.api.getSelectedNodes() || [];
    const selected = selectedNodes.map(node => node.data);
    setSelectedRows(selected);
  }, []);

  // Generate YAML from selected rows
  const generateYaml = useCallback(() => {
    const selected = gridRef.current?.api.getSelectedRows() || [];
    
    // @ts-ignore
    if (typeof vscode !== 'undefined') {
      // @ts-ignore
      vscode.postMessage({
        command: 'generateYaml',
        selectedItems: selected,
      });
    }
  }, []);

  // Expand/Collapse all
  const expandAll = useCallback(() => {
    gridRef.current?.api.expandAll();
  }, []);

  const collapseAll = useCallback(() => {
    gridRef.current?.api.collapseAll();
  }, []);

  // Select/Deselect all
  const selectAll = useCallback(() => {
    gridRef.current?.api.selectAll();
  }, []);

  const deselectAll = useCallback(() => {
    gridRef.current?.api.deselectAll();
  }, []);

  // Listen for messages from extension
  React.useEffect(() => {
    const handleMessage = (event: MessageEvent) => {
      const message = event.data;
      
      switch (message.command) {
        case 'schemaFetched':
          setRowData(transformSchemaToTreeData(message.schema));
          break;
        case 'yamlGenerated':
          // Display YAML in a preview panel
          console.log('Generated YAML:', message.yaml);
          break;
      }
    };

    window.addEventListener('message', handleMessage);
    return () => window.removeEventListener('message', handleMessage);
  }, []);

  return (
    <div style={{ height: '100vh', display: 'flex', flexDirection: 'column' }}>
      {/* Toolbar */}
      <div style={{
        padding: '10px',
        borderBottom: '1px solid var(--vscode-panel-border)',
        display: 'flex',
        gap: '10px',
      }}>
        <button onClick={fetchSchema} className="vscode-button">
          🔄 Fetch Schema from Org
        </button>
        <button onClick={expandAll} className="vscode-button">
          ➕ Expand All
        </button>
        <button onClick={collapseAll} className="vscode-button">
          ➖ Collapse All
        </button>
        <button onClick={selectAll} className="vscode-button">
          ☑ Select All
        </button>
        <button onClick={deselectAll} className="vscode-button">
          ☐ Deselect All
        </button>
        <button
          onClick={generateYaml}
          disabled={selectedRows.length === 0}
          className="vscode-button primary"
        >
          📄 Generate YAML ({selectedRows.length} selected)
        </button>
      </div>

      {/* AG-Grid */}
      <div className="ag-theme-vscode-dark" style={{ flex: 1 }}>
        <AgGridReact
          ref={gridRef}
          columnDefs={columnDefs}
          rowData={rowData}
          defaultColDef={defaultColDef}
          autoGroupColumnDef={autoGroupColumnDef}
          treeData={true}
          animateRows={true}
          groupDefaultExpanded={0}
          getDataPath={(data: SchemaNode) => data.orgHierarchy}
          rowSelection="multiple"
          suppressRowClickSelection={true}
          onSelectionChanged={onSelectionChanged}
          enableCellTextSelection={true}
          ensureDomOrder={true}
        />
      </div>

      {/* Status Bar */}
      <div style={{
        padding: '5px 10px',
        borderTop: '1px solid var(--vscode-panel-border)',
        fontSize: '12px',
        color: 'var(--vscode-descriptionForeground)',
      }}>
        {selectedRows.length} item(s) selected | {rowData.length} total objects
      </div>
    </div>
  );
};

// Transform Salesforce schema to tree data format
function transformSchemaToTreeData(schema: any): SchemaNode[] {
  return schema.objects.map((obj: any) => ({
    id: obj.apiName,
    objectName: obj.label,
    apiName: obj.apiName,
    type: obj.type,
    label: obj.label,
    dataType: 'Object',
    orgHierarchy: [obj.apiName],
    children: obj.fields?.map((field: any) => ({
      id: `${obj.apiName}.${field.apiName}`,
      objectName: field.label,
      apiName: field.apiName,
      type: field.type,
      label: field.label,
      required: field.required,
      length: field.length,
      dataType: 'Field',
      orgHierarchy: [obj.apiName, field.apiName],
    })) || [],
  }));
}
```

### **4. Styles** (`webview.css`)

```css
:root {
  --vscode-button-background: var(--vscode-button-background);
  --vscode-button-foreground: var(--vscode-button-foreground);
}

.vscode-button {
  background: var(--vscode-button-background);
  color: var(--vscode-button-foreground);
  border: none;
  padding: 6px 14px;
  cursor: pointer;
  font-size: 13px;
  border-radius: 2px;
}

.vscode-button:hover {
  background: var(--vscode-button-hoverBackground);
}

.vscode-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.vscode-button.primary {
  background: var(--vscode-button-secondaryBackground);
  color: var(--vscode-button-secondaryForeground);
}

/* AG-Grid VSCode theme customization */
.ag-theme-vscode-dark {
  --ag-background-color: var(--vscode-editor-background);
  --ag-foreground-color: var(--vscode-editor-foreground);
  --ag-header-background-color: var(--vscode-editorGroupHeader-tabsBackground);
  --ag-odd-row-background-color: var(--vscode-editor-background);
  --ag-row-hover-color: var(--vscode-list-hoverBackground);
  --ag-selected-row-background-color: var(--vscode-list-activeSelectionBackground);
  --ag-border-color: var(--vscode-panel-border);
}
```

### **5. Package.json**

```json
{
  "name": "salesforce-yaml-generator",
  "displayName": "Salesforce YAML Schema Generator",
  "description": "Generate YAML schemas from Salesforce objects",
  "version": "1.0.0",
  "engines": {
    "vscode": "^1.85.0"
  },
  "categories": ["Other"],
  "activationEvents": ["onCommand:salesforce-yaml-generator.openSchemaGenerator"],
  "main": "./out/extension.js",
  "contributes": {
    "commands": [
      {
        "command": "salesforce-yaml-generator.openSchemaGenerator",
        "title": "Salesforce: Open Schema Generator",
        "category": "Salesforce"
      }
    ]
  },
  "dependencies": {
    "ag-grid-community": "^31.0.0",
    "ag-grid-react": "^31.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "js-yaml": "^4.1.0"
  },
  "devDependencies": {
    "@types/vscode": "^1.85.0",
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "typescript": "^5.3.0",
    "webpack": "^5.89.0",
    "webpack-cli": "^5.1.0"
  }
}
```

---

## **WHY NOT TABULATOR?**

While Tabulator is good, it has limitations for VSCode extensions:

❌ **No native tree data** - Uses row grouping (less intuitive)  
❌ **Larger bundle** - 150KB vs AG-Grid's better features  
❌ **jQuery heritage** - Not as React-friendly  
❌ **Less TypeScript support** - Types are community-maintained  
❌ **No VSCode theme** - Manual CSS customization needed  

---

## **FINAL RECOMMENDATION**

**Use AG-Grid Community** with the architecture above. It gives you:

1. ✅ Professional VSCode extension UX
2. ✅ Tree + table in one component
3. ✅ Checkbox selection with parent-child logic
4. ✅ Export, filter, search built-in
5. ✅ Type-safe TypeScript development
6. ✅ Free and open source

This is the **industry standard** for complex data grids in enterprise applications.
