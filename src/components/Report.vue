<script setup>
defineProps({
  title: String,
  datas: Array,
  columns: Array,
  type: String,
  description: String,
})
const emit = defineEmits(['onHover']);

// From I2aelba - https://stackoverflow.com/questions/15900485/correct-way-to-convert-size-in-bytes-to-kb-mb-gb-in-javascript
function formatBytes(bytes) {
  const k = 1024;
  const sizes = ['MiB', 'GiB', 'TiB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return `${parseFloat((bytes / Math.pow(k, i)).toFixed(2))} ${sizes[i - 2]}`;
}

function hoverTR(e) {
  let tr = e.target.parentElement;
  while (tr.tagName !== 'TR') {
    tr = tr.parentElement;
    if (!tr) return;
  }
  const attribute = tr.getAttribute('name');
  if (attribute) emit('onHover', attribute);
}
</script>

<template>
  <div class="report">
    <h2>{{ title }}</h2>
    <p>{{ description }} <slot /></p>
    <table>
      <tr>
        <th v-for="(column, index) in columns">{{ column }}</th>
      </tr>
      <tr v-for="(dataRow, index) in datas" :name="dataRow[0]" @mouseover="hoverTR">
        <td v-if="type === 'number'" v-for="(data, index) in dataRow" :class="(index === 1 && dataRow.length > 2 && dataRow[1] > dataRow[2]) && 'bold'">
          {{ data }} 
          {{ parseInt(data) >= 134217728 ? `(${formatBytes(data)})` : '' }}
        </td>
        <td v-else-if="type === 'boolean'" v-for="(data, index) in dataRow">
          <div :class="index === 1 ? data ? 'true' : 'false' : ''">
            <img v-if="index === 1" :src="`/${data}.png`" width="10" />
            {{ data }}
          </div>
        </td>
        <td v-else-if="type === 'string'" v-for="(data, index) in dataRow" :class="index === 0 && 'bold'">
          {{ data }}
        </td>
      </tr>
    </table>
  </div>
</template>

<style scoped>
.report {
  display: flex;
  flex-direction: column;
  align-items: left;
}
p {
  width: 50vw;
  margin: 20px 0;
}
th, td {
  text-align: left;
}
th {
  background-color: #04AA6D;
  color: white;
  padding: 5px 10px;
}
td {
  padding: 5px 10px;
}
td.bold {
  font-weight: bold;
}
tr {
  transition: .5s;
}
tr:nth-child(even) {
  background-color: #f2f2f2;
}
tr:hover {
  background-color: #c8fae8;
  transition: 0s;
}
table {
  border-spacing: 0;
}
table tr:last-child td:first-child {
  border-bottom-left-radius: 10px;
}
table tr:last-child td:last-child {
  border-bottom-right-radius: 10px;
}
table tr:first-child th:first-child {
  border-top-left-radius: 10px;
}
table tr:first-child th:last-child {
  border-top-right-radius: 10px;
}
.true {
  color: #04AA6D;
}
.false {
  color: #d17171;
}
@media only screen and (max-width: 600px) {
  p {
    width: 80vw;
  }
}
</style>
