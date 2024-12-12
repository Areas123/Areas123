<template>
  <div class="x-table-wrapper">
    <div
      class="virtual-table"
      @scroll="onScroll"
      :style="{ height: containerHeight + 'px', overflow: 'auto' }"
    >
      <div
        class="virtual-table-header"
        :style="{ position: 'sticky', top: '0', zIndex: '1' }"
      >
        <template v-if="typeIndex">
          <div class="virtual-table-header-cell">
            <div style="text-align:center;" class="header-content">序号</div>
          </div>
        </template>
        <template v-for="header in tableHeader">
          <div
            v-if="header.show !== false"
            :key="header.name"
            class="virtual-table-header-cell"
            :style="{
              textAlign: header.align || 'center',
              width: header.width ? header.width + 'px' : 'auto',
              maxWidth: header.maxWidth ? header.maxWidth + 'px' : 'none',
              minWidth: header.minWidth ? header.minWidth + 'px' : 'auto',
            }"
          >
            <div
              class="header-content"
              :style="{
                justifyContent:
                  header.align === 'right'
                    ? 'flex-end'
                    : header.align === 'left'
                    ? 'flex-start'
                    : 'center',
              }"
            >
              {{ header.label }}
              <div v-if="header.sortable" class="sort-icons">
                <div
                  class="sort-asc"
                  @click="sortData(header.name, 'asc')"
                  :class="
                    sortKey === header.name && sortOrder === 'asc'
                      ? 'isActive'
                      : ''
                  "
                ></div>
                <div
                  class="sort-desc"
                  :class="
                    sortKey === header.name && sortOrder === 'desc'
                      ? 'isActive'
                      : ''
                  "
                  @click="sortData(header.name, 'desc')"
                ></div>
              </div>
            </div>
          </div>
        </template>
      </div>
      <div :style="{ height: totalHeight + 'px', position: 'relative' }">
        <template v-for="(row, index) in visibleData">
          <div
            :key="row[keyField] || index"
            :style="{
              position: 'absolute',
              height: rowHeight + 'px',
              lineHeight: rowHeight + 'px',
              top: (startIndex + index) * rowHeight + 'px',
              width: '100%',
            }"
            class="virtual-table-row"
          >
            <template v-if="typeIndex">
              <div class="virtual-table-cell">
                {{ startIndex + index + 1 }}
              </div>
            </template>
            <template v-for="header in tableHeader">
              <div
                v-if="header.show !== false"
                :key="header.name"
                class="virtual-table-cell"
                :class="{isClick:header.method && typeof header.method === 'function'}"
                :style="{
                  textAlign: header.align || 'center',
                  width: header.width ? header.width + 'px' : 'auto',
                  maxWidth: header.maxWidth ? header.maxWidth + 'px' : 'none',
                  minWidth: header.minWidth ? header.minWidth + 'px' : 'auto',
                }"
                v-html="
                  header.formatter ? header.formatter(row,startIndex + index) : row[header.name]
                "
                @click="header.method && typeof header.method === 'function' ? header.method(row,startIndex + index) : null"
              ></div>
            </template>
          </div>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "VirtualTable",
  props: {
    list: {
      type: Array,
      required: true,
    },
    rowHeight: {
      type: Number,
      default: 40,
    },
    containerHeight: {
      type: Number,
      default: 500,
    },
    keyField: {
      type: String,
      default: "id",
    },
    tableHeader: {
      type: Array,
      default: () => [],
    },
    typeIndex: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      startIndex: 0,
      endIndex: 0,
      sortedList: [],
      sortOrder: null,
      sortKey: null,
      tableHeaderHeight: 0,
    };
  },
  watch: {
    list: {
      handler(newValue) {
        this.sortedList = [...newValue];
        this.updateVisibleRange();
      },
      deep: true,
      immediate: true,
    },
  },
  computed: {
    totalHeight() {
      return this.sortedList.length * this.rowHeight;
    },
    visibleData() {
      return this.sortedList.slice(this.startIndex, this.endIndex);
    },
  },
  mounted() {
    this.tableHeaderHeight = this.$el.querySelector(
      ".virtual-table-header"
    ).offsetHeight;
  },
  methods: {
    onScroll(event) {
      const scrollTop = event.target.scrollTop;
      const visibleCount = Math.ceil(this.containerHeight / this.rowHeight);
      this.startIndex = Math.floor(scrollTop / this.rowHeight);
      const offset = this.startIndex > 0 ? this.tableHeaderHeight : 0;
      this.startIndex = Math.floor((scrollTop - offset) / this.rowHeight);
      if (this.startIndex < 0) this.startIndex = 0;

      this.endIndex = this.startIndex + visibleCount;
    },
    sortData(key, order) {
      this.sortKey = key;
      this.sortOrder = order;
      const sortMethod = this.tableHeader.find(
        (header) => header.name === key
      )?.sortMethod;
      if (sortMethod && typeof sortMethod === "function") {
        this.sortedList = [...this.list].sort((a, b) => {
          let c =  sortMethod(a, b)
          if (order === "asc") {
            return c > 0 ? 1:-1
          }
          return c > 0 ? -1 : 1
        });
      } else {
        this.sortedList = [...this.list].sort((a, b) => {
          if (order === "asc") {
            return a[key] > b[key] ? 1 : -1;
          } else {
            return a[key] < b[key] ? 1 : -1;
          }
        });
      }
    },
    updateVisibleRange() {
      const visibleCount = Math.ceil(this.containerHeight / this.rowHeight);
      this.endIndex = this.startIndex + visibleCount;
    },
  },
};
</script>

<style>
.x-table-wrapper {
  position: relative;
  border: 1px solid #dfe6ec;
  overflow: auto;
}
.virtual-table-header {
  display: flex;
  
  font-weight: bold;
  white-space: nowrap;
}
.virtual-table-header-cell {
  flex: 1;
  padding: 8px;
  box-sizing: border-box;
  text-align: left;
  background-color: #f8f8f8;
  border-bottom: 1px solid #dfe6ec;
}
.header-content {
  display: flex;
  align-items: center;
  height: 100%;
}
.sort-icons {
  display: flex;
  flex-direction: column;
  margin-left: 8px;
  position: relative;
}
.sort-asc,
.sort-desc {
  width: 0;
  height: 0;
  border: 5px solid transparent;
  cursor: pointer;
  margin-left: 5px;
}
.sort-asc {
  border-bottom-color: #c0c4cc;
  margin-bottom: 2px;
}
.sort-desc {
  border-top-color: #c0c4cc;
  margin-top: 2px;
}
.sort-asc.isActive {
  border-bottom-color: #409eff;
}
.sort-desc.isActive {
  border-top-color: #409eff;
}
.virtual-table {
  position: relative;
  overflow-y: auto;
  white-space: nowrap;
}
.virtual-table-row {
  display: flex;
  align-items: center;
  
}
.virtual-table-cell {
  flex: 1;
  padding: 0 8px;
  box-sizing: border-box;
  border-bottom: 1px solid #dfe6ec;
  cursor: default;
  height: inherit;
  line-height: inherit;
}
.virtual-table-cell.isClick{
  cursor: pointer;
}
</style>
