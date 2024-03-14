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

    <div class="table-panel">
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
          <template v-if="column !== 'id'">
            <input type="text" class="column-input" :value="columns[index]" @input="updateColumn($event, index)">
          </template>
          <template v-else>
            <div class="column-input">{{ columns[index] }}</div>
          </template>
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
          <template v-if="column !== 'id'">
            <input type="text" class="column-input" :value="columns[index]" @input="updateColumn($event, index)">
          </template>
          <template v-else>
            <div class="column-input">{{ columns[index] }}</div>
          </template>
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
          <div class="ellipsis-container-desktop" v-if="dragRowIndex === null || dragRowIndex === rowIndex">
            <img src="@/images/ellipsis.svg" class="ellipsis-icon">
          </div>
          <div class="ellipsis-container-mobile" @click="showDeleteMenu">
              <p>Действие</p>
              <img src="@/images/ellipsis.svg" class="ellipsis-icon">
          </div>
        </div>

        <div v-for="(column, colIndex) in columns" :key="colIndex" class="table-cell-desktop" :data-index="colIndex" :style="columnStyles[colIndex]" v-show="columnVisibility[colIndex]">
          <template v-if="column !== 'id'">
            <input type="text" v-model="row[column]" class="row-input" :placeholder="column">
          </template>
          <template v-else>
            <div class="indexation">
              <img src="@/images/burger-icon.svg" alt="">
              <div>{{ row[column] }}</div>
            </div>
          </template>
        </div>
        <div v-for="(column, colIndex) in columns" :key="colIndex" class="table-cell-mobile" :data-index="colIndex">
          <template v-if="column !== 'id'">
            <p>{{columns[colIndex]}}</p>
            <input type="text" v-model="row[column]" class="row-input" :placeholder="column">
          </template>
          <template v-else>
            <p class="mobile-row">{{columns[colIndex]}}</p>
            <div class="indexation">
              <img src="@/images/burger-icon.svg" alt="">
              <div>{{ row[column] }}</div>
            </div>
          </template>
        </div>
        <div class="delete-menu" v-show="isMobileMenu">
          <span @click.stop="deleteRow(rowIndex)" class="delete-button">Удалить</span>
        </div>
      </div>
      <div class="delete-modal-overlay" v-show="isMobileMenu" @click.stop="closeMobileMenu"></div>
    </div>

     <div class="modal-overlay" v-show="isModal" @click="closeModal"></div>

    <div class="modal-window" v-show="isModal">
      <div class="settings" @click="openSubModal('columns')">
        <span>Отображение столбцов</span>
        <img src="@/images/arrow-right.svg" alt="">
      </div>
    </div>

    <div class="sub-modal-overlay" v-show="isSubModalOpen" @click.stop="closeSubModal"></div>

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

    <button class="button-style" @click="saveChanges">Сохранить изменения</button>
  </div>
</template>

<script>
import '@/styles/global.css';
export default {
  data() {
    return {
      columns: ['id', 'Наименование единицы', 'Цена', 'Название товара', 'Итого', 'Вес', 'Автор', 'Описание', 'Время', 'Дата'],
      rows: [],
      columnVisibility: Array(10).fill(true),
      selectAllChecked: false,
      resizingColumnIndex: null,
      isMobileMenu: false,
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
        'Наименование единицы': '', 
        'Цена': '0',
        'Название товара': '',
        'Итого': '0',
        'Вес': '0',
        'Автор': '',
        'Описание': '',
        'Время': '',
        'Дата': ''
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
    deleteRow(rowIndex) {
      this.rows.splice(rowIndex, 1);

      this.rows.forEach((row, index) => {
        row.id = index + 1;
      });

      this.isMobileMenu = false;
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
        const cellElement = document.querySelector(`.table-cell-desktop[data-index="${this.resizingColumnIndex}"]`);
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
    showDeleteMenu(){
      this.isMobileMenu = !this.isMobileMenu;
    },
    closeMobileMenu(){
      this.isMobileMenu = false;
    }
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
  width: 108rem;
  height: fit-content;
  padding: 0.625rem 1.5625rem 0.625rem 0.625rem;
  background-color: #eeeff1;
  border-radius: 0.625rem;
  box-shadow: 0 0.3125rem 1.25rem 0 rgba(0, 0, 0, 0.07);
  border: solid 0.0625rem var(--pale-grey);
}

.controls {
  margin-bottom: 0.625rem;
  display: flex;
  flex-direction: row;
  width: 100%;

  div.right-panel{
    display: flex;
    justify-content: flex-end;
    gap: 1.25rem;
    width: 50%;

    img{
      width: 0.9375rem;
      height: 0.9375rem;
      object-fit: contain;
    }
  }
  div.left-panel{
    display: flex;
    justify-content: flex-start;
    gap: 1.25rem;
    width: 50%;

    button.button-style{
      width: 9.3125rem;
      height: 3.4375rem;
      font-size: 0.875rem;
    }

    .add-row{
      width: 9.125rem;
      height: 2.1875rem;
      padding: 0.625rem 0.9375rem 0.625rem 0.625rem;
      border-radius: 0.3125rem;
      background-color: #2f8cff;
      cursor: pointer;

      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: center;
      gap: 0.4375rem;

      span{
        font-size: 0.875rem;
        width: fit-content;
        font-weight: normal;
        font-stretch: normal;
        font-style: normal;
        line-height: normal;
        letter-spacing: normal;
        color: #fff;
      }

      img{
        width: 0.6875rem;
        height: 0.6875rem;
        margin: 0.125rem 0.4375rem 0.125rem 0;
        border-radius: 6.25rem;
      }
    }

    .add-column{
      width: 10rem;
      height: 2.1875rem;
      padding: 0.625rem 0.9375rem 0.625rem 0.625rem;
      border-radius: 0.3125rem;
      background-color: #2f8cff;
      cursor: pointer;

      display: flex;
      flex-direction: row;
      justify-content: center;
      align-items: center;
      gap: 0.4375rem;

      span{
        font-size: 0.875rem;
        width: fit-content;
        font-weight: normal;
        font-stretch: normal;
        font-style: normal;
        line-height: normal;
        letter-spacing: normal;
        color: #fff;
      }

      img{
        width: 0.6875rem;
        height: 0.6875rem;
        margin: 0.125rem 0.4375rem 0.125rem 0;
        border-radius: 6.25rem;
      }
    }
  }
}

.indexation{
  display: flex;
  flex-direction: row;
  gap: 0.625rem;
}

.modal-window{
    display: flex;
    flex-direction: column;
    position: absolute;
    right: 0;
    z-index: 1000;
    top: 4.75rem;
    width: 11.1875rem;
    height: fit-content;
    margin: 0.3125rem 0.9375rem 3.875rem 51.375rem;
    padding: 0.4375rem;
    border-radius: 0.3125rem;
    box-shadow: 0 0 0.1875rem 0 #000, inset 0 0.0625rem 0.125rem 0 rgba(255, 255, 255, 0.5);
    background-color: #fff;
    gap:0.625rem;

    .settings{
      display: flex;
      flex-direction: row;
      justify-content: center;
      gap: 0.625rem;
    }
}

.sub-modal{
  display: flex;
  flex-direction: column;
  position: absolute;
  right: 0;
  z-index: 999;
  top: 4.75rem;
  width: 11.1875rem;
  height: fit-content;
  margin: 0.3125rem 0.9375rem 3.875rem 51.375rem;
  padding: 0.4375rem;
  border-radius: 0.3125rem;
  box-shadow: 0 0 0.1875rem #000, inset 0 0.0625rem 0.125rem 0 rgba(255, 255, 255, 0.5);
  background-color: #fff;
  gap:0.625rem;

  .settings{
    display: flex;
    flex-direction: row;
    justify-content: center;
    gap: 0.625rem;

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

.mobile-row{
  display: none;
}

.delete-menu{
  display: none;
}

.table-cell-mobile{
  display: none;
}

@media (max-width: 768px) {
  .table{
    width: 100%;
    padding: 0.9375rem 0.9375rem 1.5625rem;
  }
  button.button-style{
    width: 100%;
    height: 3.4375rem;
  }
  .table-header{
    display: none;
  }
  .table-cell-desktop{
    display: none;
  }
  .table-cell-mobile{
    display: block;

    .row-input{
      height: 2.1815rem;
    }
  }
  .delete-modal-overlay{
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 5;
  }
  .tabel-panel{
    border-radius: 0.625rem;
    width: 100%;
  }
  .table-row{
    display: block;
    margin-bottom: 1.5625rem;
    padding: 0.625rem;
    background: white;
    border-radius: 0.625rem;
  }
  .row-checkbox{
    display: none;
  }
  .delete-menu{
    display: block;
    width: 11.1875rem;
    height: 1.8125rem;
    z-index: 6;
    position: absolute;
    border: 1px solid black;
    padding: 0.4375rem 1.2125rem 0.4375rem 0.4375rem;
    top: 0.625rem;
    left: 0;
    background: white;
    border-radius: 0.625rem;
    .delete-button{
      color: #ae0a0a;
      font-weight: normal;
      font-stretch: normal;
      font-style: normal;
      line-height: normal;
      letter-spacing: normal;
      font-size: 1.125rem;
    }
  }
  .indexation{
    margin-bottom: 1.5625rem;
  }
  .mobile-row{
    display: block;
    margin-block-start: 0;
  }
  .controls{
     div.right-panel{
        display: none;
     }
     div.left-panel{
        width: 100%;
        gap: 0.625rem;
        justify-content: space-between;

        input{
          width: 7.1875rem;
        }

        .add-column{
          width: 5rem;
        }
        .add-row{
          width: 5.125rem;
        }

        .button-style{
          display: none;
        }
        
     }
  }

  .ellipsis-block{
    width: 0.625rem;
    height: 0.625rem;
    margin-bottom: 3.225rem;
  }

  .ellipsis-container-desktop{
    display: none;
  }
  .ellipsis-container-mobile{
    display: block;

    img{
      width: 0.6875rem;
      height: 0.6875rem;
    }

    p{
      margin-block-start: 0;
      margin-block-end: 0.225rem;
    }
  }

  .table-cell{
    margin-top: 1.625px;
    width: 100%;
  }


  .table-row:hover .ellipsis-container-desktop{
    display: none;
  }
  

}
</style>