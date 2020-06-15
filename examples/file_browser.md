# File Browser Example

A simple file browser example built with [`regular-table`](https://github.com/jpmorganchase/regular-table).

```html
<regular-table id="regularTable"></regular-table>
```

# File System

Use a 2D Array, row oriented.

```javascript
const DATA = [];
```

Generate directory contents on the fly, given a path.

```javascript
function* generateDirContents(root = []) {
    for (let i = 0; i < 5; i++) {
        const path = [...root, `Dir_${i}`];
        const row = [new Date(), "directory", true];
        yield {path, row, is_open: false};
    }
    for (let i = 0; i < 5; i++) {
        const path = [...root, `File_${i}.txt`];
        const row = [new Date(), "file", true];
        yield {path, row};
    }
}
```

Toggle open or closed a path.

```javascript
function toggleDir(y) {
    const path = DATA[y].path;
    if (DATA[y].is_open) {
        while (DATA[y + 1] && DATA[y + 1].path.length > path.length) {
            DATA.splice(y + 1, 1);
        }
    } else {
        DATA.splice(y + 1, 0, ...Array.from(generateDirContents(DATA[y].path)));
    }
    DATA[y].is_open = !DATA[y].is_open;
}
```

# Virtual Data Model

Need to transpose the data set, because it is row-oriented - `regular-table`
expects column-oriented data.

```javascript
function transpose(m) {
    return m.length === 0 ? [] : m[0].map((x, i) => m.map((x) => x[i]));
}
```

Otherwise standard.

```javascript
function dataListener(x0, y0, x1, y1) {
    return {
        num_rows: DATA.length,
        num_columns: DATA[0].row.length,
        row_headers: DATA.slice(y0, y1).map((z) => _tree_header(z.path)),
        column_headers: [["modified"], ["kind"], ["writable"]],
        data: transpose(DATA.slice(y0, y1).map(({row}) => row.slice(x0, x1))),
    };
}
```

# Custom Style

Clipping the intermediate elements of the tree makes them 0 with `<th>`, which
makes the row groups appear "tree" like.

```javascript
function _tree_header(path) {
    return path
        .slice(0, path.length - 1)
        .fill("")
        .concat(path[path.length - 1]);
}
```

Folder Icons styles applied as classes.

```javascript
function styleListener() {
    for (const td of window.regularTable.querySelectorAll("tbody th")) {
        const meta = window.regularTable.getMeta(td);
        td.classList.toggle("fb-directory", meta.value && DATA[meta.y].row[1] === "directory");
        td.classList.toggle("fb-open", meta.value && DATA[meta.y].is_open);
    }
}
```

# CSS

Icons

```css
tbody th.fb-directory:before {
    font-family: "Material Icons";
    content: "folder ";
}
tbody th.fb-directory.fb-open:before {
    content: "folder_open ";
}
tbody th:not(.fb-directory):not(:empty) {
    padding-left: 20px;
}
```

Set dimensions of "tree" structure.

```css
tbody th:empty {
    min-width: 20px;
    max-width: 20px;
}
```

And some 

```css
table {
    user-select: none;
}
```

# UI

Click to toggle

```javascript
function mousedownListener() {
    if (event.target.tagName === "TH") {
        const meta = window.regularTable.getMeta(event.target);
        toggleDir(meta.y);
        window.regularTable.resetAutoSize();
        window.regularTable.draw();
    }
}
```

# Main

```javascript
function init() {
    DATA.splice(0, 0, ...Array.from(generateDirContents())); // = Array.from(generateDirContents());
    window.regularTable.setDataListener(dataListener);
    window.regularTable.addStyleListener(styleListener);
    window.regularTable.addEventListener("mousedown", mousedownListener);
    window.regularTable.addEventListener("scroll", () => window.regularTable.resetAutoSize());
    window.regularTable.draw();
}
```

```html
<script>window.addEventListener("load", () => init())</script>
```

# Appendix (Dependencies)

```html
<script src="/dist/umd/regular-table.js"></script>
<link rel='stylesheet' href="/dist/css/material.css">
```

```block
license: apache-2.0
```
