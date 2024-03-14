<template>
  <div class="table">
    <div class="controls">
      <div class="left-panel">
        <input type="text" v-model="newColumnName" placeholder="Enter column name">
        <div class="add-column" @click="addColumn">
          <img src="@/images/plus-icon.svg" alt="">
          <span>Добавить столбец</span>
        </div>
        <div class="add-row" @click="addRow">
          <img src="@/images/plus-icon.svg" alt="">
          <span>Добавить строку</span>
        </div>
        <button class="button-style" @click="toggleEditMode">Переключить режим</button>
        <button class="button-style" @click="deleteSelectedRows" v-if="selectedRows.size > 0">Удалить строку</button>
      </div>
      <div class="right-panel">
        <span class="pointer" @click="saveChanges">Сохранить изменения</span>
        <img src="@/images/settings-icon.svg" alt="" @click="openModal">
      </div>
    </div>

    <div class="table">
      <div class="table-header" v-if="editMode === 'drag'">
        <div class="table-header-item">
          <input type="checkbox" @change="switchSelectedAllRows">
        </div>
        <div
          v-for="(column, index) in columns"
          :key="index"
          class="table-header-item handle"
          :class="{ 'dragging': dragColumnIndex === index, 'droppable': dropColumnIndex === index }"
          :data-index="index"
          :style="columnStyles[index]" 
          draggable="true"
          @dragstart="dragStartColumn(index)"
          @dragover.prevent
          @drop="dropColumn(index)"
          v-show="columnVisibility[index]"
        >
          <div class="column-input">{{columns[index]}}</div>
        </div>
      </div>
      <div class="table-header" v-else>
        <div
          v-for="(column, index) in columns"
          :key="index"
          class="table-header-item resize"
          :data-index="index"

          :style="columnStyles[index]" 
          @mousedown="resizeColumnStart(index, $event)"
        >
          <input type="text" v-model="columns[index]" class="column-input">
        </div>
      </div>

      <div
        v-for="(row, rowIndex) in rows"
        :key="row.id"
        :class="{
          'table-row': true,
          'selected': selectedRows.has(rowIndex),
          'dragging': dragRowIndex === rowIndex,
          'droppable': dropRowIndex === rowIndex
        }"
        draggable="true"
        :data-index="rowIndex"
        @dragstart="dragStartRow(rowIndex)"
        @dragover.prevent
        @drop="dropRow(rowIndex)"
      >

        <div
          v-if="isDragging && dropRowIndex !== null"
          class="table-row-placeholder"
          :style="{ top: dropRowIndex * rowHeight + 'px' }"
        ></div>

        <input 
          type="checkbox" 
          :checked="isSelected(rowIndex)" 
          @change="toggleSelection(rowIndex)" 
          class="row-checkbox"
        >

        <div class="ellipsis-block">
          <div class="ellipsis-container" v-if="dragRowIndex === null || dragRowIndex === rowIndex">
            <img src="@/images/ellipsis.svg" class="ellipsis-icon">
          </div>
        </div>
        <div v-for="(column, colIndex) in columns" :key="colIndex" class="table-cell" :data-index="colIndex" :style="columnStyles[colIndex]" v-show="columnVisibility[colIndex]">
          <input type="text" v-model="row[column]" class="row-input" :placeholder="column">
        </div>
      </div>
    </div>

     <div class="modal-overlay" v-show="isModal" @click="closeModal"></div>

    <div class="modal-window" v-show="isModal">
      <div class="settings" @click="openSubModal('columns')">
        <span>Отображение столбцов</span>
        <img src="@/images/arrow-right.svg" alt="">
      </div>
    </div>

    <div class="sub-modal-overlay" v-show="isSubModalOpen" @click="closeSubModal"></div>

    <div class="sub-modal" v-show="isSubModalOpen">
      <div class="settings" @click="closeSubModal">
        <img src="@/images/arrow-right.svg" alt="">
        <span>Отображение столбцов</span>
      </div>
      <div v-for="(column, index) in columns" :key="index">
        <input type="checkbox" v-model="columnVisibility[index]">
        <label>{{ column }}</label>
      </div>
    </div>
  </div>
</template>

<script>
import '@/styles/global.css';
export default {
  data() {
    return {
      columns: ['id', 'name', 'price', 'capacity', 'product name', 'weight', 'author_name', 'description', 'time', 'time log'],
      rows: [],
      columnVisibility: Array(10).fill(true),
      selectAllChecked: false,
      resizingColumnIndex: null,
      startX: 0,
      startWidth: 0,
      lastId: 0,
      newColumnName: '',
      isDragging: false,
      dragRowIndex: null,
      dropRowIndex: null,
      dragColumnIndex: null,
      dropColumnIndex: null,
      selectedRows: new Set(),
      rowHeight: 10,
      editMode: 'drag',
      isModal: false,
      isSubModalOpen: false,
      subModalContent: null
    };
  },
  computed:{
    columnWidths() {
      if (!this.columns) return []; 
      return this.columns.map(() => 189);
    },
    columnStyles() {
      return this.columns.map((column, index) => ({
        width: this.columnWidths[index] + 'px'
      }));
    },
  },
  methods: {
    addColumn() {
      const newColumnName = this.newColumnName || 'New Column';
      this.columns.push(newColumnName);
      this.columnVisibility.push(true);
      this.newColumnName = ''; 
    },
    addRow(){
      this.rows.push({
        id: this.generateId(),
        name: '', 
        price: '',
        capacity: '',
        'product name': '',
        weight: '',
        author_name: '',
        description: '',
        time: '',
        'time log': ''
      });
    },
    dragStartColumn(index) {
      this.dragColumnIndex = index;
    },
    dropColumn(index) {
      this.dropColumnIndex = index;
      setTimeout(() => {
        const draggedColumn = this.columns.splice(this.dragColumnIndex, 1)[0];
        this.columns.splice(this.dropColumnIndex, 0, draggedColumn);
        this.dragColumnIndex = null;
        this.dropColumnIndex = null;
      }, 300); 
    },
    dragStartRow(rowIndex) {
      this.dragRowIndex = rowIndex;
      this.isDragging = true;
    },
    dropRow(dropRowIndex) {
      this.dropRowIndex = dropRowIndex;
      setTimeout(() => {
        const draggedRow = this.rows.splice(this.dragRowIndex, 1)[0];
        this.rows.splice(this.dropRowIndex, 0, draggedRow);
       
        this.rows.forEach((row, index) => {
          row.id = index + 1;
        });
        this.dragRowIndex = null;
        this.dropRowIndex = null;
        this.isDragging = true;
      }, 300); 
    },
    toggleSelection(rowIndex) {
      if (this.isSelected(rowIndex)) {
        this.selectedRows.delete(rowIndex);
        console.log(this.selectedRows)
      } else {
        this.selectedRows.add(rowIndex);
        console.log(this.selectedRows)
      }
    },
    isSelected(rowIndex) {
      console.log(this.selectedRows.has(rowIndex))
      return this.selectedRows.has(rowIndex);
    },
    switchSelectedAllRows(){
      this.selectAllChecked = !this.selectAllChecked;
      this.selectAllRows();
    },
    selectAllRows() {
      if (this.selectAllChecked) {
        this.rows.forEach((row, index) => {
          this.selectedRows.add(index);
        });
      } else {
        this.selectedRows.clear();
      }
    },
    deleteSelectedRows() {
      const selectedRowsArray = [...this.selectedRows].sort((a, b) => b - a);
      selectedRowsArray.forEach(index => {
        this.rows.splice(index, 1);
      });
      this.selectedRows.clear();
      
      this.rows.forEach((row, index) => {
        row.id = index + 1;
      });
    },
    generateId() {
      return this.rows.length + 1; 
    },
    resizeColumnStart(index, event) {
      console.log("resizeColumnStart");
      this.resizingColumnIndex = index;
      this.startX = event.pageX;
      this.startWidth = this.columnWidths[index];
      document.addEventListener('mousemove', this.resizeColumn);
      document.addEventListener('mouseup', this.resizeColumnEnd);
    },
    resizeColumn(event) {
        console.log("resizeColumn");
        const deltaX = event.pageX - this.startX;
        const newWidth = Math.min(window.innerWidth, this.startWidth + deltaX); 
        console.log("DeltaX:", deltaX, "StartX:", this.startX, "NewWidth:", newWidth);

        
        this.columnWidths[this.resizingColumnIndex] = newWidth;
        console.log("New param of columnWidth:", this.columnWidths);

        
        const columnElement = document.querySelector(`.table-header-item[data-index="${this.resizingColumnIndex}"]`);

        
        columnElement.style.width = newWidth + 'px';

        
        const cellElement = document.querySelector(`.table-cell[data-index="${this.resizingColumnIndex}"]`);

        
        cellElement.style.width = newWidth + 'px';

        
        this.startX = event.pageX;
    },
    resizeColumnEnd() {
      console.log("resizeColumnEnd");
      document.removeEventListener('mousemove', this.resizeColumn);
      document.removeEventListener('mouseup', this.resizeColumnEnd);
      this.resizingColumnIndex = null;
    },
    toggleEditMode() {
      
      this.editMode = this.editMode === 'drag' ? 'resize' : 'drag';
    },
    openModal() {
      this.isModal = !this.isModal;
    },
    openSubModal(content) {
      
      this.isModal = false;
      this.isSubModalOpen = true;
      this.subModalContent = content;
    },
    closeSubModal() {
      
      this.isSubModalOpen = false;
      this.isModal = true;
    },
    closeModal() {
      this.isModal = false;
    },
    saveChanges() {
      
      localStorage.setItem('columns', JSON.stringify(this.columns));
      localStorage.setItem('rows', JSON.stringify(this.rows));
    },
  },
  mounted() {
    
    const storedColumns = localStorage.getItem('columns');
    if (storedColumns) {
      this.columns = JSON.parse(storedColumns);
    }
    const storedRows = localStorage.getItem('rows');
    if (storedRows) {
      this.rows = JSON.parse(storedRows);
    }
  }
};
</script>

<style lang="scss" scoped>

.table{
  width: 1728px;
  height: fit-content;
  padding: 10px 25px 10px 10px;
  background-color: #fbfcfd;
  border-radius: 10px;
  box-shadow: 0 5px 20px 0 rgba(0, 0, 0, 0.07);
  border: solid 1px var(--pale-grey);
}

.controls {
  margin-bottom: 10px;
  display: flex;
  flex-direction: row;
  width: 100%;

  div.right-panel{
    display: flex;
    justify-content: flex-end;
    gap: 20px;
    width: 50%;

    img{
      width: 15px;
      height: 15px;
      object-fit: contain;
    }
  }
  div.left-panel{
    display: flex;
    justify-content: flex-start;
    gap: 20px;
    width: 50%;

    button.button-style{
      width: 149px;
      height: 55px;
      font-size: 14px;
    }

    .add-row{
      width: 146px;
      height: 35px;
      padding: 10px 15px 10px 10px;
      border-radius: 5px;
      background-color: #2f8cff;
      cursor: pointer;

      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: center;
      gap: 7px;

      span{
        font-size: 14px;
        width: fit-content;
        font-weight: normal;
        font-stretch: normal;
        font-style: normal;
        line-height: normal;
        letter-spacing: normal;
        color: #fff;
      }

      img{
        width: 11px;
        height: 11px;
        margin: 2px 7px 2px 0;
        border-radius: 100px;
      }
    }

    .add-column{
      width: 160px;
      height: 35px;
      padding: 10px 15px 10px 10px;
      border-radius: 5px;
      background-color: #2f8cff;
      cursor: pointer;

      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: center;
      gap: 7px;

      span{
        font-size: 14px;
        width: fit-content;
        font-weight: normal;
        font-stretch: normal;
        font-style: normal;
        line-height: normal;
        letter-spacing: normal;
        color: #fff;
      }

      img{
        width: 11px;
        height: 11px;
        margin: 2px 7px 2px 0;
        border-radius: 100px;
      }
    }
  }
}

.modal-window{
    display: flex;
    flex-direction: column;
    position: absolute;
    right: 0;
    z-index: 1000;
    top: 30px;
    width: 179px;
    height: fit-content;
    margin: 5px 15px 62px 822px;
    padding: 7px;
    border-radius: 5px;
    box-shadow: 0 0 3px 0 #000, inset 0 1px 2px 0 rgba(255, 255, 255, 0.5);
    background-color: #fff;
    gap:10px;

    .settings{
      display: flex;
      flex-direction: row;
      justify-content: center;
      gap: 10px;
    }
}

.sub-modal{
  display: flex;
  flex-direction: column;
  position: absolute;
  right: 0;
  z-index: 999;
  top: 30px;
  width: 179px;
  height: fit-content;
  margin: 5px 15px 62px 822px;
  padding: 7px;
  border-radius: 5px;
  box-shadow: 0 0 3px 0 #000, inset 0 1px 2px 0 rgba(255, 255, 255, 0.5);
  background-color: #fff;
  gap:10px;

  .settings{
    display: flex;
    flex-direction: row;
    justify-content: center;
    gap: 10px;

    img{
      transform: rotate(180deg);
    }
  }
}
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 999;
}


.sub-modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 998;
}
</style>