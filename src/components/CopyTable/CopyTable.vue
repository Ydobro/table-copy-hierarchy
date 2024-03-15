<template>
  <div ref="tableElement" class="table-responsive">
    <table class="copy-table">
      <thead class="copy-table__header">
        <tr class="copy-table__row copy-table-row">
          <th>Lane</th>
          <th v-for="field in fields" :key="field.key">{{ field.label }}</th>
          <td></td>
        </tr>
      </thead>
      <tbody id="list" class="copy-table__body">
        <tr v-for="(lane, laneIndex) in lanes" :key="lane" class="copy-table__row copy-table-row">
          <td align="center">{{ lane }}</td>
          <td
            v-for="(field, fieldIndex) in fields"
            :key="field.key"
            tabindex="0"
            class="ui-selectable"
            :class="{
              'ui-copy-selected':
                selectedItemsState?.value?.[laneIndex * fields.length + fieldIndex]
            }"
            :data-key="field.key"
          >
            <input type="text" v-model="items[laneIndex * fields.length + fieldIndex]" />
          </td>
          <td>
            <button @click="onAddFieldClick">Add Field</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted } from 'vue'
import { useEventListener } from '@/helpers/event'

import Selectable from 'selectable.js' // See: https://mobius-studios.gitbook.io/selectable

let selectable = null
const selectedItemsState = reactive([])

onMounted(() => {
  selectable = new Selectable({
    container: '#list',
    filter: '.ui-selectable',
    lasso: {
      border: '1px dashed rgba(0, 0, 0, 1)',
      backgroundColor: 'rgba(255, 255, 255, 0.4)'
    }
  })
})

onUnmounted(() => {
  selectable && selectable.destroy()
})

useEventListener(document, 'keydown', (event) => {
  // Ctrl+C or Cmd+C pressed
  if ((event.ctrlKey || event.metaKey) && event.keyCode == 67) {
    onCopyAction()
  }

  // Ctrl+V or Cmd+V pressed
  if ((event.ctrlKey || event.metaKey) && event.keyCode == 86) {
    onPasteAction()
  }
})

const onCopyAction = () => {
  selectedItemsState.value = selectable.getItems().map((el) => el.selected)
  selectable.clear()
}

const onPasteAction = () => {
  console.log('selectedItemsState', selectedItemsState.value)
}

const tableElement = ref(null)

const lanes = Array.from({ length: 8 }, (_, i) => i + 1)
const fields = reactive([
  { key: 'name', label: 'Name' },
  { key: 'description', label: 'Description' },
  { key: 'type', label: 'Type' },
  { key: 'reagent', label: 'Reagent' },
  { key: 'sample', label: 'Sample' }
])

const items = reactive(Array.from(lanes.length * fields.length))

let newFieldKey = 0
const onAddFieldClick = () => {
  selectedItemsState.value = []

  fields.push({ key: `field_${newFieldKey}`, label: `Field ${newFieldKey + 1}` })
  setTimeout(() => {
    tableElement.value.scrollLeft = tableElement.value.scrollWidth
    selectable && selectable.update()
  })

  newFieldKey++
}
</script>

<style lang="scss" scoped></style>

<style scoped>
.table-responsive {
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}

.copy-table th,
.copy-table td {
  padding: 15px;
  border: 1px solid var(--color-cell-border, var(--color-border));
  background-color: var(--color-cell-background, var(--color-background));
}

.copy-table th:first-child,
.copy-table td:first-child {
  position: sticky;
  left: 0;
}

.copy-table th:last-child,
.copy-table td:last-child {
  position: sticky;
  right: 0;
}

.ui-selectable {
  transition: 0.3s all;
}

.ui-selected,
.ui-selecting {
  --color-cell-border: #86b7fe;
  --color-cell-background: #86b7fe10;
}

.ui-copy-selected {
  --color-cell-border: #ffc107;
  --color-cell-background: #ffc10710;
}

.ui-selectable:focus {
  outline: none;
}

/** Controls */
input {
  width: 100%;
  min-height: 34px;
  width: 150px;
  background-color: var(--color-background-soft);
  color: var(--color-text);
  padding: 4px 12px;
  border: 1px solid var(--color-border);
  transition: 0.3s all;
}

input:hover {
  border-color: var(--color-border-hover);
}

input:focus {
  outline: none;
  border-color: #86b7fe;
  box-shadow: 0 0 0 0.25rem rgba(13, 110, 253, 0.25);
}

button {
  width: 100px;
  text-align: center;
  min-height: 34px;
  padding: 4px 12px;
  border: 1px solid var(--color-border);
  background-color: var(--color-background-soft);
  color: var(--color-text);
  border: 1px solid var(--color-border);
  transition: 0.3s all;
  cursor: pointer;
}

button:hover {
  border-color: var(--color-border-hover);
}
</style>
