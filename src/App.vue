<template>
  <div class="app">
    <div class="container">
      <div class="project_name">
        <textarea
          v-model="projectName"
          :ref="'headerTextarea0'"
          @input="textareaResize(0)"
          class="project_name_textarea"
          placeholder="Укажите название проекта"
        />
      </div>
      <div class="search">
        <div class="search__header">Поиск</div>
        <div><input v-model="search" class="search__input" /></div>
        <div @click="saveTasksToFile()" class="save-load__text">
          &nbsp;&nbsp;&nbsp;Save
        </div>
        <div @click="loadTasksFromFile()" class="save-load__text">
          &nbsp;&nbsp;&nbsp;Load
        </div>
      </div>

      <!-- #########################   Задача  ############################## -->

      <Container
        @drop="onDrop"
        drag-handle-selector=".drag_handle_element"
        id="task-container"
      >
        <Draggable
          v-for="currentTask in filteredTasks"
          :key="currentTask.taskId"
        >
          <div class="list-item">
            <div
              @mouseover="subTaskMouseOver(currentTask)"
              @mouseleave="subTaskMouseLeave(currentTask)"
              v-bind:style="{
                background: currentTask.taskHovered ? '#f7f7f7' : 'white',
              }"
              class="list-item__header"
              :class="{ drag_handle_element: !currentTask.editReady }"
            >
              <div
                @click="
                  currentTask.taskOpened = !currentTask.taskOpened;
                  $nextTick(function () {
                    resizeAllTitles();
                  });
                "
                class="list-item__arrow"
              >
                <img
                  v-if="currentTask.taskOpened"
                  src="./assets/arrow_drop_down_black_24dp.svg"
                />
                <img
                  v-if="!currentTask.taskOpened"
                  src="./assets/arrow_right_black_24dp.svg"
                />
              </div>
              <div
                @click="currentTask.taskDone = !currentTask.taskDone"
                class="list-item__circle"
              >
                <img
                  v-if="currentTask.taskDone"
                  src="./assets/circle_green_24dp.svg"
                />
                <img
                  v-if="!currentTask.taskDone"
                  src="./assets/circle_black_24dp.svg"
                />
              </div>

              <div class="list-item__header_text">
                <textarea
                  v-model="currentTask.taskTitle"
                  @input="textareaResize(currentTask.taskId)"
                  @focusout="currentTask.editReady = false"
                  @click="textareaHeaderClick(currentTask)"
                  :ref="`headerTextarea${currentTask.taskId}`"
                  :readonly="!currentTask.editReady"
                  v-bind:style="{
                    background: currentTask.taskHovered ? '#f7f7f7' : 'white',
                  }"
                  class="list-item__header_textarea"
                  placeholder="Укажите заголовок"
                />
              </div>

              <div class="list-item__nav_placeholder">
                <div
                  class="list-item__nav"
                  v-if="currentTask.taskHovered"
                  @click="showSubMenu(currentTask, $event.target, 'task')"
                >
                  <img src="./assets/dots_three_circle_vertical.svg" />
                </div>
              </div>
              <div
                v-if="currentTask.subMenuShown"
                @mouseleave="menuMouseLeave(currentTask)"
                class="list-item__submenu"
                v-bind:style="{
                  top: currentTask.subMenuY + 20 + 'px',
                  left: currentTask.subMenuX - 184 + 'px',
                }"
              >
                <div
                  @click="addSubTask(currentTask)"
                  class="list-item__submenu_item"
                >
                  Добавить подзадачу
                </div>
                <div
                  @click="shareTask(currentTask)"
                  class="list-item__submenu_item"
                >
                  Поделиться задачей
                </div>

                <div
                  @click="deleteTask(currentTask)"
                  class="list-item__submenu_item"
                >
                  Удалить задачу
                </div>
              </div>
            </div>

            <!-- #########################   Подзадачи  ############################## -->

            <div v-if="currentTask.taskOpened">
              <Container
                group-name="subTasks"
                @drop="(e) => onSubTaskDrop(currentTask.taskId, e)"
                :get-child-payload="getTaskPayload(currentTask.taskId)"
                drag-handle-selector=".drag_handle_element"
                id="subtask-container"
              >
                <Draggable
                  v-for="currentSubTask in currentTask.taskSubs"
                  :key="currentSubTask.taskId"
                >
                  <div
                    @mouseover="subTaskMouseOver(currentSubTask)"
                    @mouseleave="subTaskMouseLeave(currentSubTask)"
                    v-bind:style="{
                      background: currentSubTask.taskHovered
                        ? '#f7f7f7'
                        : 'white',
                    }"
                    :class="{ drag_handle_element: !currentSubTask.editReady }"
                  >
                    <div class="list-item__options">
                      <div
                        @click="subTaskDoneClicked(currentSubTask)"
                        class="list-item__circle"
                      >
                        <img
                          v-if="currentSubTask.taskDone"
                          src="./assets/circle_green_24dp.svg"
                        />
                        <img
                          v-if="!currentSubTask.taskDone"
                          src="./assets/circle_black_24dp.svg"
                        />
                      </div>
                      <div class="list-item__option__text">
                        <textarea
                          v-model="currentSubTask.taskTitle"
                          @input="textareaResize(currentSubTask.taskId)"
                          @focusout="currentSubTask.editReady = false"
                          @click="textareaHeaderClick(currentSubTask)"
                          :ref="`headerTextarea${currentSubTask.taskId}`"
                          :readonly="!currentSubTask.editReady"
                          v-bind:style="{
                            background: currentSubTask.taskHovered
                              ? '#f7f7f7'
                              : 'white',
                          }"
                          class="list-item__option_textarea"
                          placeholder="Укажите заголовок"
                        />
                      </div>
                      <div class="list-item__nav_placeholder">
                        <div
                          class="list-item__nav"
                          v-if="currentSubTask.taskHovered"
                          @click="
                            showSubMenu(
                              currentSubTask,
                              $event.target,
                              'subtask'
                            )
                          "
                        >
                          <img src="./assets/dots_three_circle_vertical.svg" />
                        </div>
                      </div>
                      <div
                        v-if="currentSubTask.subMenuShown"
                        @mouseleave="menuMouseLeave(currentSubTask)"
                        class="list-item__submenu"
                        v-bind:style="{
                          top: currentSubTask.subMenuY + 0 + 'px',
                          left: currentSubTask.subMenuX - 190 + 'px',
                        }"
                      >
                        <div
                          @click="deleteSubTask(currentTask, currentSubTask)"
                          class="list-item__submenu_item"
                        >
                          Удалить подзадачу
                        </div>
                      </div>
                    </div>
                  </div>
                </Draggable>
              </Container>
            </div>
          </div>
        </Draggable>
      </Container>

      <div class="button">
        <button @click="addTask()" class="button__add-note">
          + Новая запись
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { Container, Draggable } from "vue-dndrop";
import { applyDrag } from "./utils";
import { tzData } from "./tz";

export default {
  name: "vue-todo-list-hard",

  components: { Container, Draggable },

  data() {
    return {
      // Импортируется из tz.js в created()
      listData: [],

      // Шаблон задачи для создания новых задач. Также используется для хранения последнего taskId
      taskTemplate: {
        taskId: 1,
        taskTitle: "",
        taskTitleHeight: 0,
        taskDone: false,
        taskOpened: false,
        taskHovered: false,
        subMenuShown: false,
        subMenuX: 0,
        subMenuY: 0,
        editReady: false,

        taskSubs: [],
      },

      projectName: "ТЗ тестового задания",
      isMobileDevice: false,
      search: "",
      broadcastChannel: new BroadcastChannel("TasksChannel"),
      haveTaskToReceive: false,
      taskToReceive: {},
      loadFileDialogOpened: false,
    };
  },

  created() {
    // Пытаемся загрузить данные из Session Storage.
    this.getDataFromSessionStorage();

    if (!this.listData.length) {
      /* Данные из Session Storage не загрузились
         Инициализируем переменные данными из внешнего файла с пунктами ТЗ.
         Последний элемент tzData - название проекта, он не нужен для listData */
      this.projectName = tzData.pop().projectName;

      this.listData = tzData.slice(0);
      this.taskTemplate.taskId = this.getLastTaskID();
    }

    // Обработчик resize окна браузера. Когда меняется размер окна браузера, будет пересчитываться
    // высота всех textarea.
    window.addEventListener("resize", this.resizeAllTitles);
  },

  mounted() {
    // Определяем, ПК у нас или мобильное устройство с тачскрином
    this.isMobileDevice = "ontouchstart" in document.documentElement;

    this.saveDataToSessionStorage();

    // Канал для передачи задач между вкладками
    this.broadcastChannel.addEventListener("message", (e) => {
      // Ставим флаг о том, что из другой вкладки передана задача.
      this.haveTaskToReceive = 1;

      // e.data - это JSON c задачей
      this.taskToReceive = e.data;
      this.addSharedTask(JSON.parse(e.data));
    });

    /* Сразу после начальной загрузки страницы, textarea иногда содержит некорректное
       значение scrollheight. Через некоторое время scrollheight волшебным образом
       становится корректным. Выжидаем некоторое время, чтобы дождаться этого.
       Вроде есть вариант попробовать вместо textarea <span contenteditable></span> */
    setTimeout(this.resizeAllTitles, 100);
  },

  computed: {
    filteredTasks() {
      return this.listData.filter((task) => {
        return this.findInTask(task, this.search);
      });
    },
  },

  watch: {
    // Изменение высоты блоков при работе фильтра
    filteredTasks() {
      this.$nextTick(() => {
        this.resizeAllTitles();
      });
    },

    listData: {
      // Deep Watch. При большом количестве данных могут быть проблемы с производительностью.
      // Сохраняем данные в Session storage при каждом изменении. Ввели одну букву - сохранили.
      handler() {
        this.saveDataToSessionStorage();
      },
      deep: true,
    },
    projectName: {
      handler() {
        this.saveDataToSessionStorage();
      },
      deep: true,
    },
  },

  methods: {
    onDrop(dropResult) {
      this.listData = applyDrag(this.listData, dropResult);
    },

    // Возвращает последний использованный ID в списке задач
    getLastTaskID() {
      let lastID = 1;
      this.listData.forEach((task) => {
        // Сравниваем с taskID самой задачи
        lastID = task.taskId > lastID ? task.taskId : lastID;
        // Углубляемся в подзадачи
        if (task.taskSubs.length) {
          task.taskSubs.forEach((subTask) => {
            lastID = subTask.taskId > lastID ? subTask.taskId : lastID;
          });
        }
      });
      return lastID;
    },
    saveDataToSessionStorage() {
      sessionStorage.setItem("listData", JSON.stringify(this.listData));
      sessionStorage.setItem("projectName", JSON.stringify(this.projectName));
    },

    getDataFromSessionStorage() {
      const storedListData = JSON.parse(sessionStorage.getItem("listData"));
      const storedProjectName = JSON.parse(
        sessionStorage.getItem("projectName")
      );

      // Если были данные в sessionStorage
      if (storedListData) {
        this.listData = storedListData;
        this.taskTemplate.taskId = this.getLastTaskID();
      }
      if (storedProjectName) {
        this.projectName = storedProjectName;
      }
    },

    // Возвращает true, если строка есть в задаче или в любой её подзадаче
    findInTask(task, stringToFind) {
      const upperCaseTitle = task.taskTitle.toUpperCase();
      const upperCaseDataToFind = stringToFind.toUpperCase();

      if (upperCaseTitle.includes(upperCaseDataToFind) == true) {
        return true;
      }

      if (task.taskSubs.length) {
        for (let i = 0; i < task.taskSubs.length; i++) {
          if (this.findInTask(task.taskSubs[i], stringToFind)) {
            return true;
          }
        }
      }
      return false;
    },

    showSubMenu(currentSubTask, subMenuButton, taskLevel) {
      let dndTaskParent = subMenuButton.parentNode;

      /* Подменю позиционируется в задачах относительно первого контейнера DnD
         с id task-container.*/
      if (taskLevel === "task") {
        while (dndTaskParent.id != "task-container") {
          dndTaskParent = dndTaskParent.parentNode;
        }
      }
      /* Подменю позиционируется в подзадачах относительно вложенного контейнера DnD
         с id subtask-container.*/
      if (taskLevel === "subtask") {
        while (dndTaskParent.id != "subtask-container") {
          dndTaskParent = dndTaskParent.parentNode;
        }
      }
      currentSubTask.subMenuX =
        subMenuButton.getBoundingClientRect().x -
        dndTaskParent.getBoundingClientRect().x;

      currentSubTask.subMenuY =
        subMenuButton.getBoundingClientRect().y -
        dndTaskParent.getBoundingClientRect().y;
      /*
        Проблема: меню подзадачи не получается позиционировать красиво как меню задачи, т.к.
        оно выйдет за пределы родительского элемента и порежется снизу (для последней подзадачи).
        Попробовать определять последнюю подзадачу и отображать для неё подменю повыше? 
      */
      currentSubTask.subMenuShown = true;
    },

    subTaskMouseOver(currentSubTask) {
      if (!this.isMobileDevice) {
        currentSubTask.taskHovered = true;
      }
    },

    subTaskMouseLeave(currentSubTask) {
      currentSubTask.taskHovered = false;
      currentSubTask.subMenuShown = false;
    },

    menuMouseLeave(currentTask) {
      currentTask.subMenuShown = false;
    },

    subTaskDoneClicked(currentSubTask) {
      currentSubTask.taskDone = !currentSubTask.taskDone;
      currentSubTask.taskHovered = false;
    },

    addTask() {
      this.taskTemplate.taskId++;
      const newTask = this.copyObject(this.taskTemplate);
      this.listData.push(newTask);
    },

    deleteTask(taskToRemove) {
      this.listData = this.listData.filter((t) => t !== taskToRemove);
    },

    addSubTask(currentTask) {
      this.taskTemplate.taskId++;
      const newSubTask = this.copyObject(this.taskTemplate);
      currentTask.taskSubs.push(newSubTask);
    },

    deleteSubTask(taskContainer, subTaskToRemove) {
      taskContainer.taskSubs = taskContainer.taskSubs.filter(
        (t) => t !== subTaskToRemove
      );
    },

    /* Такой вот метод копирования объекта по значению */
    copyObject(objectToCopy) {
      return JSON.parse(JSON.stringify(objectToCopy));
    },

    textareaResize(taskId) {
      const elementRef = this.$refs["headerTextarea" + taskId];

      if (taskId > 0) {
        if (elementRef && elementRef.length) {
          const element = elementRef[0];

          let event = new Event("resize");
          element.dispatchEvent(event);
          element.style.height = "18px";
          element.style.height = element.scrollHeight + "px";
        }
      }

      // Обрабатываем отдельно textarea для названия проекта
      if (taskId == 0) {
        // Реф для названия проекта не обрабатывается как остальные,
        // для него не формируется elementRef[0]
        const element = elementRef;
        element.style.height = "18px";
        element.style.height = element.scrollHeight - 10 + "px";
      }
    },
    resizeAllTitles() {
      for (let i = 0; i <= this.taskTemplate.taskId; i++) {
        this.textareaResize(i);
      }
    },

    textareaHeaderClick(currentSubTask) {
      // Для мобильного устройства второй клик включает редактирование, для ПК - первый.

      if (!this.isMobileDevice) {
        currentSubTask.editReady = true;
      } else {
        currentSubTask.taskHovered
          ? (currentSubTask.editReady = true)
          : (currentSubTask.taskHovered = true);
      }
    },

    saveTasksToFile() {
      // Делаем копию массива с задачами
      const dataToSave = this.listData.slice(0);
      // Добавляем название проекта к массиву задач для сохранения одним массивом
      dataToSave.push({ projectName: this.projectName });
      // Делаем ссылку на BLOB
      const a = document.createElement("a");
      a.href = URL.createObjectURL(
        new Blob([JSON.stringify(dataToSave, null, 2)], {
          type: "text/plain",
        })
      );
      a.setAttribute("download", this.projectName + ".txt");
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
    },

    loadTasksFromFile() {
      const input = document.createElement("input");
      input.type = "file";
      input.style.display = "none";
      this.loadFileDialogOpened = true;

      // Обработчик change стартует после выбора файла
      input.addEventListener("change", (event) => {
        const file = event.target.files[0];
        const reader = new FileReader();
        reader.readAsText(file);

        reader.onload = () => {
          this.listData = JSON.parse(reader.result);
          // Последним элементом хранится название проекта, вынимаем его
          this.projectName = this.listData.pop().projectName;

          this.taskTemplate.taskId = this.getLastTaskID();
          this.resizeAllTitles();
        };
      });
      document.body.appendChild(input);

      // Клик показывает окно выбора файла
      input.click();

      // Вешаем обработчик на фокус основного окна. Когда закроется диалоговое окно выбора файла,
      // фокус вернется в основное окно. Такой вот способ отловить закрытие диалогового окна :)
      window.addEventListener(
        "focus",
        () => {
          if (this.loadFileDialogOpened) {
            document.body.removeChild(input);
            this.loadFileDialogOpened = false;
          }
        },
        { once: true }
      );
    },

    shareTask(taskToShare) {
      // Отправляем задачу в Broadcast Channel
      this.broadcastChannel.postMessage(JSON.stringify(taskToShare));
      taskToShare.subMenuShown = false;
      taskToShare.taskHovered = false;
    },

    addSharedTask(taskToAdd) {
      // Принимаем задачу от другой вкладки. Главная проблема - поменять Id
      taskToAdd.taskId = ++this.taskTemplate.taskId;
      taskToAdd = this.resetTaskVisualAttributes(taskToAdd);

      taskToAdd.taskSubs.forEach((subTask) => {
        subTask.taskId = ++this.taskTemplate.taskId;

        subTask = this.resetTaskVisualAttributes(subTask);
      });

      const newTask = this.copyObject(taskToAdd);
      this.listData.push(newTask);
    },

    resetTaskVisualAttributes(taskToReset) {
      taskToReset.taskHovered = false;
      taskToReset.subMenuShown = false;
      taskToReset.subMenuX = 0;
      taskToReset.subMenuY = 0;
      taskToReset.editReady = false;
      return taskToReset;
    },
    onSubTaskDrop(taskId, dropResult) {
      // DnD drop, сработает для каждого draggable элемента, т.е. для каждой основной задачи
      if (dropResult.removedIndex !== null || dropResult.addedIndex !== null) {
        const listDataCopy = this.listData.slice(0);
        const task = listDataCopy.filter((p) => p.taskId === taskId)[0];
        const taskIndex = listDataCopy.indexOf(task);
        /* Нашли индекс в массиве затронутого элемента, делаем копию затронутого элемента массива */
        const changedTask = this.copyObject(task);
        //Передаем массив подзадач затронутого элемента в applyDrag
        changedTask.taskSubs = applyDrag(changedTask.taskSubs, dropResult);
        // Удаляем старую задачу из списка и заменяем её изменённой версией
        listDataCopy.splice(taskIndex, 1, changedTask);

        this.listData = this.copyObject(listDataCopy);
      }
    },
    getTaskPayload(taskId) {
      // Здесь используется замыкание. Фактически возвращается функция, в которой уже
      // хранится taskId и которой передается индекс подзадачи, которую она в итоге
      // и возвращает.
      return (index) => {
        return this.listData.filter((p) => p.taskId === taskId)[0].taskSubs[
          index
        ];
      };
    },
  },
};
</script>
