<script lang="ts">
export default {
  name: "TodoApp",
  data() {
    return {
      inputText: "",
      toggleStatus: "all",
      todoData: [] as { txt: string; id: number; checked: string }[],
      tabs: [
        { label: "全部", value: "all" },
        { label: "待完成", value: "work" },
        { label: "已完成", value: "done" },
      ],
    };
  },
  computed: {
    filteredTodos(): { txt: string; id: number; checked: string }[] {
      if (this.toggleStatus === "work") {
        return this.todoData.filter((i) => i.checked === "");
      } else if (this.toggleStatus === "done") {
        return this.todoData.filter((i) => i.checked === "checked");
      }
      return this.todoData;
    },
    remaining(): number {
      return this.todoData.filter((i) => i.checked === "").length;
    },
  },
  methods: {
    addTodo(): void {
      if (this.inputText.trim() === "") return;
      this.todoData.unshift({
        txt: this.inputText,
        id: Date.now(),
        checked: "",
      });
      this.inputText = "";
    },
    deleteTodo(id: number): void {
      this.todoData = this.todoData.filter((i) => i.id !== id);
    },
    deleteCompleted(): void {
      this.todoData = this.todoData.filter((i) => i.checked !== "checked");
    },
    changeTab(tabValue: string): void {
      this.toggleStatus = tabValue;
    },
  },
  mounted() {
    // 初始化時讀取 localStorage
    const savedData = localStorage.getItem("todoData");
    if (savedData) {
      try {
        this.todoData = JSON.parse(savedData);
      } catch (e) {
        console.error("讀取 localStorage 失敗", e);
      }
    }
  },
  watch: {
    // 深度監聽 todoData，確保每次變動都儲存
    todoData: {
      handler(newValue) {
        localStorage.setItem("todoData", JSON.stringify(newValue));
      },
      deep: true,
    },
  },
};
</script>

<template>
  <div class="container">
    <h1>TODO LIST</h1>

    <!-- 輸入區 -->
    <div class="card input">
      <input
        type="text"
        placeholder="請輸入待辦事項"
        v-model="inputText"
        @keypress.enter="addTodo"
      />
      <a href="#" class="btn_add" @click.prevent="addTodo">+</a>
    </div>

    <!-- 清單區 -->
    <div class="card card_list">
      <!-- Tabs -->
      <ul class="tab">
        <li
          v-for="tab in tabs"
          :key="tab.value"
          :data-tab="tab.value"
          :class="{ active: toggleStatus === tab.value }"
          @click="changeTab(tab.value)"
        >
          {{ tab.label }}
        </li>
      </ul>

      <div class="cart_content">
        <!-- 代辦清單 -->
        <ul class="list">
          <li v-for="item in filteredTodos" :key="item.id">
            <label class="checkbox">
              <input
                type="checkbox"
                v-model="item.checked"
                true-value="checked"
                false-value=""
              />
              <span :class="{ 'line-through': item.checked === 'checked' }">
                {{ item.txt }}
              </span>
            </label>
            <a href="#" class="delete" @click.prevent="deleteTodo(item.id)"></a>
          </li>
        </ul>

        <!-- 底部 -->
        <div class="list_footer">
          <p><span>{{ remaining }}</span> 個待完成項目</p>
          <a href="#" @click.prevent="deleteCompleted">清除已完成項目</a>
        </div>
      </div>
    </div>
  </div>
</template>
<style>
@import url("https://fonts.googleapis.com/earlyaccess/notosanstc.css");
@import url("https://fonts.googleapis.com/css?family=Baloo+Tamma+2:600,700");

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html, body {
  height: 100%;
}

body {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  min-height: 100vh;
  font-family: "Noto Sans TC", sans-serif;
  background-image: linear-gradient(
    174deg,
    #ffd370 2%,
    #ffd370 46%,
    #ffffff 46%,
    #ffffff 100%,
    #e8e8e8 100%
  );
  background-repeat: no-repeat;
  background-size: cover;
  color: #333;
  letter-spacing: 0.07em;
}

h1 {
  text-align: center;
  font-size: 3rem;
  margin-bottom: 1.5rem;
  font-family: "Baloo Tamma 2";
  letter-spacing: 0.5rem;
  font-weight: bold;
}

/* 小螢幕時字體縮小 */
@media (max-width: 575px) {
  h1 {
    font-size: 2rem;
    margin-bottom: 1rem;
  }
}

/* Reset ul, li 預設樣式 */
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}

/* 響應式容器 */
.container {
  width: 100%;        /* 預設佔滿螢幕寬度 */
  min-width: 500px;   /* 桌面最大寬度 */
  margin: 0 auto;
  padding: 0 1rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  box-sizing: border-box;
}
@media (max-width: 575px) {
  .container {
    min-width: 65%
  }
}
/* 卡片樣式 */
.card {
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  width: 100%;
  box-sizing: border-box;
}

/* 輸入區塊 */
.input {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1rem;
  width: 100%;
  box-sizing: border-box;
}

input[type="text"] {
  flex: 1;
  width: 100%;
  border: none;
  outline: none;
  font-size: 1rem;
  padding: 0.75rem 1rem;
  color: #333;
  background: transparent;
  box-sizing: border-box;
  height: 48px;
}

.btn_add {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 48px;
  height: 48px;
  background: #333;
  color: #fff;
  font-size: 1.5rem;
  border-radius: 10px;
  text-decoration: none;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
  flex-shrink: 0;
}

/* 任務列表卡片 */
.card_list {
  display: flex;
  flex-direction: column;
  width: 100%;
}

/* tabs */
.tab {
  display: flex;
  justify-content: space-between;
  color: #9f9a91;
  font-size: 14px;
  border-bottom: 2px solid #efefef;
}

.tab li {
  flex: 1;
  text-align: center;
  padding: 1rem 0;
  cursor: pointer;
  position: relative;
}

.tab li.active {
  color: #333;
  font-weight: bold;
}

.tab li.active::after {
  content: "";
  position: absolute;
  bottom: -2px;
  left: 0;
  width: 100%;
  border-bottom: 2px solid #333;
}

.cart_content {
  padding: 0.5rem 1rem 1rem 1rem;
}

.list_footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 14px;
  padding: 1rem 0.5rem 0 0.5rem;
}

/* checkbox */
.checkbox {
  display: block;
  position: relative;
  padding-left: 44px;
  cursor: pointer;
}

.checkbox span {
  display: block;
  padding: 1rem 0;
  border-bottom: 1px solid #eee;
  line-height: 1.5;
}

.checkbox input {
  position: absolute;
  top: 0;
  left: 0;
  opacity: 0;
  cursor: pointer;
  height: 100%;
  width: 100%;
}

.checkbox span::before {
  content: "";
  position: absolute;
  left: 0.5rem;
  top: 50%;
  transform: translateY(-50%) scale(1);
  height: 20px;
  width: 20px;
  border-radius: 5px;
  border: 1px solid #333;
  transition: 0.3s ease;
}

.checkbox span::after {
  content: "";
  position: absolute;
  left: 14px;
  top: 27%;
  transform: rotate(45deg);
  height: 15px;
  width: 0.5rem;
  border-bottom: 3px solid #ffd370;
  border-right: 3px solid #ffd370;
  opacity: 0;
  transition: 0.3s ease;
}

.checkbox input:checked ~ span {
  color: #9f9a91;
  text-decoration: line-through;
}

.checkbox input:checked ~ span::before {
  border-color: transparent;
  transform: translateY(-50%) scale(0);
}

.checkbox input:checked ~ span::after {
  opacity: 1;
}

/* 📱 小螢幕優化 */
@media (max-width: 480px) {
  .input {
    padding: 0.5rem 0.75rem;
  }
  input[type="text"] {
    font-size: 0.9rem;
    height: 44px;
  }
  .btn_add {
    width: 44px;
    height: 44px;
  }
}


</style>
