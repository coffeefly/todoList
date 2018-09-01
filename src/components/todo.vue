<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <input type="text" class="todoInput" placeholder="比如：吃个晚饭" @keyup.enter="add" v-model="todoItem" />
    <div class="btn-group">
      <button class="finished" @click="updateList('finished')">已完成</button>
      <button class="unfinished" @click="updateList('unfinished')">未完成</button>
      <button class="all" @click="updateList('all')">全部</button>
    </div>
    <!-- 任务列表 -->
    <ul class="todoUl">
      <h2 v-if="filterList.length==0">当前没有任务</h2>
      <li v-for="(todo,index) in filterList" :key="todo.id" :class="{editing: todo == editedTodo }">
        <div class="view">
          <input type="checkbox" :id="todo.id" :checked="todo.finished" v-model="todo.finished" @click="todo.finished=!todo.finished" />
          <label :for="todo.id" @dblclick="edit(todo,index)" :class="{finished:todo.finished}"> {{ todo.item }}</label>
          <input type="button" class="del-btn" value="删除" @click="del(index)" />
        </div>
        <input class="edit" type="text" v-model="todo.item" v-todo-focus="todo == editedTodo" @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.esc="cancelEdit(todo)">
      </li>
    </ul>
  </div>
</template>

<script>
var storage_key = "todos";

const todostorage = {
  fetch: function() {
    var todos = JSON.parse(localStorage.getItem(storage_key) || "[]");
    todos.forEach((todo, index) => {
      todo.id = index;
    });
    localStorage.uid = todos.length;
    return todos;
  },
  save: function(todos) {
    localStorage.setItem(storage_key, JSON.stringify(todos));
  }
};
const filter = {
  all: function(list) {
    return list;
  },
  finished: function(list) {
    return list.filter(function(item) {
      return item.finished;
    });
  },
  unfinished: function(list) {
    return list.filter(function(item) {
      return !item.finished;
    });
  }
};
export default {
  name: "todoList",
  data() {
    return {
      msg: "一个 todoList 小程序",
      todoItem: "",
      todoList: todostorage.fetch(),
      visibility: "all",
      editedTodo: null
    };
  },
  computed: {
    filterList: function() {
      return filter[this.visibility](this.todoList);
    }
  },
  watch: {
    todoList: {
      handler: todoList => {
        todostorage.save(todoList);
      },
      // 深度观察
      deep: true
    }
  },
  methods: {
    add: function() {
      var value = this.todoItem && this.todoItem.trim();
      if (!value) {
        return;
      }
      this.todoList.push({
        id: todostorage.uid++,
        finished: false,
        item: this.todoItem
      });
      this.todoItem = "";
    },
    del: function(index) {
      this.todoList.splice(index, 1);
    },
    edit: function(todo, index) {
      console.log(this.beforeEditCache)
      this.beforeEditCache = todo.item;
      this.editedTodo = todo;
    },
    updateList: function(obj) {
      this.visibility = obj;
    },
    doneEdit: function(todo) {
       if (!this.editedTodo) {
        return
      }
      this.editedTodo = null
      todo.item = todo.item.trim()
      if (!todo.item) {
        this.del(todo)
      }
    },
    cancelEdit: function(todo) {
      this.editedTodo = null
      todo.item = this.beforeEditCache
    }
  },
  // a custom directive to wait for the DOM to be updated
  // before focusing on the input field.
  directives: {
    "todo-focus": function(el, binding) {
      if (binding.value) {
        el.focus();
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}

.hello {
  width: 500px;
  height: auto;
  margin: 0 auto;
}
.todoInput {
  width: 100%;
  height: 35px;
  padding: 0 10px;
  box-sizing: border-box;
}

.todoUl {
  width: 100%;
  text-align: left;
  padding: 0px;
}

.todoUl li {
  list-style: none;
  border: 1px solid #ccc;
  height: 30px;
  line-height: 30px;
  padding: 0 10px;
  color: #666;
}

.todoUl li:nth-child(n + 2) {
  border-top: 0;
}

.todoUl li label {
  cursor: pointer;
}

.todoUl li label.finished {
  text-decoration: line-through;
  color: #ccc;
}

/* 自定义checkbox */
.todoUl li input[type="checkbox"] {
  -webkit-appearance: none;
  outline: none;
  width: 20px;
  height: 20px;
  cursor: pointer;
  background: #fff;
  border: 1px solid #ccc;
  position: relative;
  vertical-align: middle;
}

.todoUl li input[type="checkbox"]:checked {
  background-color: #eee;
}
.todoUl li input[type="checkbox"]:checked::after {
  content: "\2713";
  display: block;
  width: 20px;
  height: 20px;
  position: absolute;
  left: 3px;
  line-height: 18px;
  font-size: 16px;
  color: red;
}

/* 调整按钮样式 */

.btn-group {
  text-align: right;
  margin-top: 10px;
}

button {
  display: inline-block;
  padding: 6px 12px;
  margin-bottom: 0;
  font-size: 14px;
  font-weight: 400;
  line-height: 1.42857143;
  text-align: center;
  white-space: nowrap;
  vertical-align: middle;
  cursor: pointer;
  user-select: none;
  background-image: none;
  border: 1px solid lightcyan;
  border-radius: 4px;
  color: #000;
}

button.finished {
  background-color: lightblue;
}

button.unfinished {
  background-color: lightcoral;
}

button.all {
  background-color: lightgreen;
}

.del-btn {
  float: right;
  vertical-align: middle;
  margin-top: 3px;
  cursor: pointer;
}

.edit {
  display: none;
  position: relative;
  margin: 0;
  width: 100%;
  height: 100%;
  font-size: 18px;
  font-family: inherit;
  font-weight: inherit;
  border: 0;
  color: inherit;
  border: 1px solid #999;
  box-shadow: inset 0 -1px 5px 0 rgba(0, 0, 0, 0.2);
  box-sizing: border-box;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.editing {
  padding: 0;
  border-bottom: none;
}

.editing .edit {
  display: block;
  margin-left: 11px;
}

.editing .view{
  display:none;
}
</style>
