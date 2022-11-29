<template>
  <div class="pagination">
    <ul class="pager clear" @click="onPage">
      <li
        :class="{ disabled: page <= 1 }"
        :aria-disabled="page <= 1"
        :arial-label="arialLabel(-1)"
        tabindex="0"
        class="pn prev"
        data-page="-1"
        role="button">
        <span class="arrow"></span>
      </li>
      <template v-for="(group, index) in slices">
        <li
          v-if="group[0] === -1"
          :key="'g' + index"
          :data-page="group[1]"
          :data-jumper="index"
          :arial-label="arialLabel(group[1])"
          class="pn jumper">
          <span class="dots">...</span>
        </li>
        <li
          v-for="i in group"
          v-else
          :key="'l' + i"
          :class="{ active: page === i }"
          :data-page="i"
          :arial-label="arialLabel(i)"
          class="pn"
          role="button">
          <span>{{ i }}</span>
        </li>
      </template>
      <li
        :class="{ disabled: page >= pages }"
        :aria-disabled="page >= pages"
        :arial-label="arialLabel(0)"
        tabindex="0"
        class="pn next"
        data-page="0"
        role="button">
        <span class="arrow"></span>
      </li>
    </ul>
    <div v-if="showSizes" class="page-size">
      每页显示数量
      <div class="page-select" @mouseenter="showPageList = true" @mouseleave="showPageList = false">
        {{ pageSize }}
        <div v-if="showPageList" class="select-box">
          <div v-for="(item, index) in pageSizeList" :key="index" class="seleclt-opotion" @click.stop="onSize(item)">
            {{ item }}
          </div>
        </div>
      </div>
      共{{ total }}条
    </div>
  </div>
</template>
 
<script>
export default {
  name: 'Pagination',
  props: {
    page: {
      type: Number,
      default: 1,
    },
    total: {
      type: Number,
      default: 0,
    },
    pageSize: {
      type: Number,
      default: 10,
    },
    onPageChange: {
      type: Function,
      default: () => {},
    },
    onPageSizeChange: {
      type: Function,
      default: () => {},
    },
    // 选择分页size
    showSizes: {
      type: Boolean,
      default: false,
    },
    // 页码list
    pageSizeList: {
      type: Array,
      default: [10, 20],
    },
  },
  data() {
    return {
      pages: 0,
      slices: [[1]],
      showPageList: false,
    };
  },
  watch: {
    page() {
      this.updateSlices();
    },
    total() {
      this.updateSlices();
    },
    pageSize() {
      this.updateSlices();
    },
  },
  mounted() {
    this.updateSlices();
  },
  methods: {
    arialLabel(i) {
      if (i === -1) {
        return '上一页';
      }
      if (i === 0) {
        return '下一页';
      }
      return `第${i}页`;
    },
    updateSlices() {
      const pages = (this.pages = Math.ceil(this.total / this.pageSize));
      if (pages < 5) {
        this.slices = [
          Array(pages)
            .fill(1)
            .map((o, i) => i + 1),
        ];
      } else if (this.page < 4) {
        this.slices = [[1, 2, 3], [-1, 4], [pages]];
      } else if (pages - this.page < 3) {
        this.slices = [[1], [-1, 2], [pages - 2, pages - 1, pages]];
      } else {
        this.slices = [[1], [-1, 2], [this.page - 1, this.page, this.page + 1], [-1, this.page + 2], [pages]];
      }
    },
    // 选择size
    onSize(e) {
      this.onPageSizeChange(e);
      //const page = Math.ceil(this.total / e);
      this.onPageChange(1);
    },
    onPage(e) {
      let target = e.target;
      if (target.tagName === 'SPAN') {
        target = target.parentElement;
      }
      if (target.className.indexOf('disabled') !== -1 || target.tagName !== 'LI') {
        return;
      }

      const page = +target.getAttribute('data-page');
      const jumper = target.getAttribute('data-jumper');
      console.log('page', page, 'jumper', jumper);
      if (jumper) {
        this.onPageChange(page);
      } else {
        this.onPageChange(this.calcNextPage(page));
      }
    },
    calcNextPage(page) {
      return !page ? this.page + 1 : page < 0 ? this.page - 1 : page;
    },
    showJumper(num, target) {
      if (num && num > 0 && num <= this.pages) {
        const slices = [...this.slices];
        slices[num][2] = 1;
        this.slices = slices;
        setTimeout(() => {
          target.children[1].focus();
        }, 100);
      }
    },
    onJump(e) {
      console.log(e.target.value);
      const val = +e.target.value;
      if (val && val > 0 && val <= this.pages) {
        this.onPageChange(val);
      }
    },
    onBlur(e) {
      const num = +e.target.parentNode.getAttribute('data-jumper');
      const slices = [...this.slices];
      slices[num][2] = 0;
      this.slices = slices;
    },
  },
};
</script>
<style scoped>
.pagination {
  font-size: $font-size-second;
  color: #999;
  letter-spacing: 1.8px;
  font-weight: 400;
  line-height: 40px;
  display: flex;
  margin-top: 50px;
  text-align: center;
}
.pagination > .pager {
  display: inline-block;
  white-space: nowrap;
}
.pagination .pn {
  float: left;
  cursor: pointer;
  width: 40px;
  height: 40px;
  line-height: 40px;
  margin-left: 24px;
  text-align: center;
  background: #f6f7fc;
  border-radius: 2px;
  font-family: PingFangSC-Regular;
  font-size: 18px;
  color: #999999;
  letter-spacing: 1.8px;
  font-weight: 400;
  outline: none;
}
.pagination .pn:first-child {
  margin-left: 0;
}
.pagination .pn:hover:not(.disabled) {
  background-color: #3a5ecf;
  color: #fff;
}
.pagination .pn.active.active {
  background-color: #3a5ecf;
  border-color: #3a5ecf;
  color: #fff;
}
.pagination .pn > .dots {
  display: block;
  font-weight: bold;
  line-height: 30px;
  padding-bottom: 6px;
}
.pagination .pn > input {
  color: #666;
  border: 0;
  font-family: Arial, sans-serif;
  font-size: 15px;
  max-width: 40px;
  padding: 2px 1px;
}
.pagination .pn.prev,
.pagination .pn.next {
  color: #999999;
  display: flex;
  align-items: center;
  justify-content: center;
}
.pagination .pn.prev .arrow {
  width: 8px;
  height: 8px;
  display: block;
  margin-left: 4px;
  border-left: solid 1px currentColor;
  border-top: solid 1px currentColor;
  -webkit-transform: rotate(-45deg);
  transform: rotate(-45deg);
}
.pagination .pn.next .arrow {
  width: 8px;
  height: 8px;
  display: block;
  margin-right: 4px;
  border-bottom: solid 1px currentColor;
  border-right: solid 1px currentColor;
  -webkit-transform: rotate(-45deg);
  transform: rotate(-45deg);
}
.pagination .pn.next,
.pagination .pn.prev {
  color: #fff !important;
  background: #3a5ecf;
}
.pagination .pn.disabled {
  cursor: not-allowed;
  background: #f6f7fc;
  color: #999 !important;
}
.pagination > .elevator {
  display: inline-block;
  color: #888f9c;
  font-size: 14px;
  height: 40px;
  line-height: 40px;
  margin-left: 20px;
  position: relative;
  vertical-align: top;
}
.pagination > .elevator > .pagenum {
  appearance: none;
  background: transparent;
  color: #666;
  border: 1px solid #e6e7eb;
  border-radius: 0;
  font-size: 18px;
  margin: 0 10px 0 2px;
  padding-left: 10px;
  width: 60px;
  height: 40px;
}

.pagination > .elevator > .icn {
  display: block;
  border-top: 6px solid #5c5c5c;
  border-left: 5px solid transparent;
  border-right: 5px solid transparent;
  border-bottom: none;
  pointer-events: none;
  position: absolute;
  top: 17px;
  left: 142px;
}
.pagination .page-size {
  margin-left: 30px;
  display: flex;
  align-items: center;
}
.pagination .page-size.page-select {
  height: 40px;
  background: #f6f7fc;
  min-width: 55px;
  padding: 0 8px;
  margin: 0 8px;
  border: none;
  outline: none;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  cursor: pointer;
}
.pagination .page-size.page-select::after {
  content: '';
  width: 0px;
  display: block;
  height: 0px;
  line-height: 0px;
  margin-left: 6px;
  border-top: 6px solid #d8d8d8;
  border-left: 6px solid #f6f7fc;
  border-right: 6px solid #f6f7fc;
}
.pagination .page-size.page-select.select-box {
  position: absolute;
  left: 0;
  width: 100%;
  top: 40px;
  border: 1px solid #eee;
}
.pagination .page-size.page-select.select-box.seleclt-opotion {
  padding: 0 10px;
}
.pagination .page-size.page-select.select-box.seleclt-opotion:hover {
  background: #f6f7fc;
}
ul li{
    list-style: none;
}
</style>