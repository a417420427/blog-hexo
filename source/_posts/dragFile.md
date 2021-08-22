---
title: 文件/文件夹拖拽上传实现
categories: javascript
tags: 拖拽上传 dragevent
---

## 相关事件

1. dragenter 拖拽进目标区域
2. dragover 拖拽经过目标区域
3. drop 拖拽结束

## 实现

1. 确定相关事件

```js
function dragFiles(el) {
  const dragenter = function (e) {
    // 阻止浏览器的默认事件， 不然拖拽结束后会直接在浏览器打开文件/文件夹
    e.preventDefault();
  };
  const dragover = function (e) {
    e.preventDefault();
  };
  const drop = function (e) {
    e.preventDefault();
  };

  el.addEventListener("dragenter", dragenter);
  el.addEventListener("dragover", dragover);
  el.addEventListener("drop", drop);
}
```

2. 读取文件

```js
const drop = async function (e) {
  e.preventDefault();
  const dropFiles = await readDropFiles();
};

async function readDropFiles(e) {
  // 保存读取到的文件 包括文件本身和文件路径
  const dropFiles = [];
  // 获取所有入口
  const entries = savedEntries(e);

  for (let i = 0; i < entries.length; i++) {
    const entry = entries[i];
    dropFiles = dropFiles.concat(await readEntry(entry));
  }
  return dropFiles;
}

// https://stackoverflow.com/questions/28487352/dragndrop-datatransfer-getdata-empty/28487486
// 保存入口 由于 DataTransfer 只在 drop的时间段存在, 所以需要提前收集文件信息
function savedEntries(e) {
  //const entries: FileEntry[] = []
  const items = e.dataTransfer && e.dataTransfer.items;
  if (!items) {
    return [];
  }
  return Array.from(items).map((item) => item.webkitGetAsEntry());
}

// 读取文件入口 提取文件
async function readFileEntrySync(entry) {
  return new Promise((resolve) => {
    entry.file((file) => {
      resolve(file);
    });
  });
}
// 读取文件夹入口 单独提取入口
async function readDirEntrySync(entry) {
  return new Promise((resolve) => {
    const fileEntries = [];
    const dirReader = entry.createReader();
    dirReader.readEntries((entries) => {
      entries.forEach((entry) => {
        fileEntries.push(entry);
      });
      resolve(fileEntries);
    });
  });
}

// 读取入口, 判断入口是文件还是文件夹
async function readEntry(fileEntry) {
  let files = [];
  if (fileEntry.isFile) {
    const file = await readFileEntrySync(fileEntry);
    files.push({
      file,
      fullPath: fileEntry.fullPath,
    });
  } else if (fileEntry.isDirectory) {
    const fileEntries = await readDirEntrySync(fileEntry);
    for (let i = 0; i < fileEntries.length; i++) {
      const entry = fileEntries[i];
      files = files.concat(await readEntry(entry));
    }
  }
  return files;
}
```

3. 展示文件

```js
function renderFiles(el, dropFiles) {
  const container = document.createElement("div");
  dropFiles.forEach((dropFile) => {
    const fileEle = document.createElement("div");
    fileEle.innerHTML = `文件路径: ${dropFile.fullPath}/${dropFile.file.name}`;
  });
  el.appendChild(container);
}

renderFiles(document.quertSelector(".drag-info"));
```

## [线上 demo](https://blog.zxueping.com/dist/index.html#/Sample)
