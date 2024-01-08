<template>
  <div class="upload upload-box" id="upload" @dragover.prevent @drop="onDrop">
    拖动文件夹到此处上传
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue";

const fileList = ref<any>([]);
const onDrop = (e: DragEvent) => {
  e.stopPropagation()
  e.preventDefault()
  const { dataTransfer } = e;
  if (!dataTransfer) return;
  const items: DataTransferItemList = dataTransfer.items;
  if (items.length > 1) {
    alert('只支持上传一个文件夹');
    return;
  }
  for (let i = 0; i < items.length; i++) {
    const item: FileSystemEntry | null = items[i].webkitGetAsEntry()
    if (item) {
      checkFolders(item)
    }
  }
}
const checkFolders = (item: FileSystemEntry) => {
  if (item.isDirectory) {
    fileList.value = traverseFileTree(item);
    console.log(fileList.value)
  } else {
    alert('只支持上传文件夹');
  }
}
function traverseFileTree(item: FileSystemEntry) {
  let res: Array<File> = [];
  const getFolderInfo = (item: any, path: string, res: Array<any>) => {
    if (item.isFile) {
      item.file((file: { path: string; name: string; }) => {
        file.path = path + file.name;
        res.push(file);
      });
    } else if (item.isDirectory) {
      const dirReader = item.createReader();
      dirReader.readEntries((entries: string | any[]) => {
        for (let i = 0; i < entries.length; i++) {
          getFolderInfo(entries[i], path + item.name + "/", res);
        }
      }
      );
    }
  };
  getFolderInfo(item, "", res);
  return res;
}
</script>

<style>
.upload {
  width: 200px;
  height: 200px;
  border: 1px dashed blanchedalmond;
}
</style>