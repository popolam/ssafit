<template>
  <v-row class="fill-height">
    <v-col cols="12" md="6" class="mb-4">
      <v-sheet height="64">
        <v-toolbar
          flat
          dark
        >
          <v-btn
            outlined
            class="mr-4"
            color="grey darken-2"
            @click="setToday"
          >
            Today
          </v-btn>
          <v-btn
            fab
            text
            small
            color="grey darken-2"
            @click="prev"
          >
            <v-icon small>
              arrow_back_ios
            </v-icon>
          </v-btn>
          <v-btn
            fab
            text
            small
            color="grey darken-2"
            @click="next"
          >
            <v-icon small>
              arrow_forward_ios
            </v-icon>
          </v-btn>
          <v-toolbar-title v-if="$refs.calendar">
            {{ $refs.calendar.title }}
          </v-toolbar-title>
          <v-spacer></v-spacer>
          <v-menu
            bottom
            right
          >
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                outlined
                color="grey darken-2"
                v-bind="attrs"
                v-on="on"
              >
                <span>{{ typeToLabel[type] }}</span>
                <v-icon right>
                  mdi-menu-down
                </v-icon>
              </v-btn>
            </template>
            <v-list>
              <!--
              <v-list-item @click="type = 'day'">
                <v-list-item-title>Day</v-list-item-title>
              </v-list-item>
              <v-list-item @click="type = 'week'">
                <v-list-item-title>Week</v-list-item-title>
              </v-list-item>
              -->
              <v-list-item @click="type = 'month'">
                <v-list-item-title>Month</v-list-item-title>
              </v-list-item>
              <!--
              <v-list-item @click="type = '4day'">
                <v-list-item-title>4 days</v-list-item-title>
              </v-list-item>
              -->
            </v-list>
          </v-menu>
        </v-toolbar>
      </v-sheet>
      <v-sheet height="600">
        <v-calendar
          ref="calendar"
          v-model="focus"
          color="primary"
          :events="events"
          :event-color="getEventColor"
          :type="type"
          @click:event="showEvent"
          @click:more="viewDay"
          @click:date="viewDay"
          dark
        ></v-calendar>
          <!-- @change="updateRange" -->
        <v-menu
          v-model="selectedOpen"
          :close-on-content-click="false"
          :activator="selectedElement"
          offset-x
        >
          <v-card
            color="grey lighten-4"
            min-width="350px"
            flat
          >
            <v-toolbar
              :color="selectedEvent.color"
              dark
            >
              <v-btn icon>
                <v-icon>mdi-pencil</v-icon>
              </v-btn>
              <v-toolbar-title v-html="selectedEvent.name"></v-toolbar-title>
              <v-spacer></v-spacer>
              <v-btn icon>
                <v-icon>mdi-heart</v-icon>
              </v-btn>
              <v-btn icon>
                <v-icon>mdi-dots-vertical</v-icon>
              </v-btn>
            </v-toolbar>
            <v-card-text>
              <span v-html="selectedEvent.details"></span>
            </v-card-text>
            <v-card-actions>
              <v-btn
                text
                color="secondary"
                @click="selectedOpen = false"
              >
                Cancel
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-menu>
      </v-sheet>
    </v-col>
    <v-col cols="12" md="6" class="mb-4">
      <div class="container">
        <h2>{{focus}} 목표</h2>
        <div v-for="(todo, index) in todos" :key="index" :id="index">
          <v-layout ma-3 row justify-space-between>
          <div class="click-done" :class="{'is-done': (todo.todoDone == 1)}" v-if="todo.todoDate == focus" @click="doneTodo(todo)" >{{todo.todoContent}}</div>
          <button @click="deleteTodo(todo)" v-if="todo.todoDate == focus"><v-icon style="color: white; margin-right: 100px;">delete</v-icon></button>
          </v-layout>
        </div>
        <div>
          <h3>운동 추가</h3>
          <input type="text" v-model.trim="newTodo" @keyup.enter="setTodo" ref="todoInput" style="background-color: white; padding: 10px;">
        </div>
      </div>
    </v-col>
  </v-row>
</template>

<script>
  import {mapState} from 'vuex';

  export default {
    data() {
      return {
        focus: '',
        type: 'month',
        typeToLabel: {
          month: 'Month',
          week: 'Week',
          day: 'Day',
          '4day': '4 Days',
        },
        selectedEvent: {},
        selectedElement: null,
        selectedOpen: false,
        events: [],
        colors: ['blue', 'indigo', 'deep-purple', 'cyan', 'green', 'orange', 'grey darken-1'],
        names: ['Meeting', 'Holiday', 'PTO', 'Travel', 'Event', 'Birthday', 'Conference', 'Party'],
        newTodo: '',
      }
    },
    computed: {
      ...mapState([
        "todos", "user"
      ])
    },
    mounted () {
      this.$refs.calendar.checkChange()

      let date = new Date();
      let year = date.getFullYear();
      let month = ("0" + (1 + date.getMonth())).slice(-2);
      let day = ("0" + date.getDate()).slice(-2);
      this.focus = year + "-" + month + "-" + day;

      this.$store.dispatch("getTodos", this.focus);
    },
    methods: {
      viewDay ({ date }) {
        this.focus = date
        this.type = 'month'
        this.$store.dispatch("getTodos", this.focus);
      },
      getEventColor (event) {
        return event.color
      },
      setToday () {
        let date = new Date();
        let year = date.getFullYear();
        let month = ("0" + (1 + date.getMonth())).slice(-2);
        let day = ("0" + date.getDate()).slice(-2);
        this.focus = year + "-" + month + "-" + day;

        this.$store.dispatch("getTodos", this.focus);
      },
      prev () {
        this.$refs.calendar.prev()
      },
      next () {
        this.$refs.calendar.next()
      },
      showEvent ({ nativeEvent, event }) {
        const open = () => {
          this.selectedEvent = event
          this.selectedElement = nativeEvent.target
          requestAnimationFrame(() => requestAnimationFrame(() => this.selectedOpen = true))
        }

        if (this.selectedOpen) {
          this.selectedOpen = false
          requestAnimationFrame(() => requestAnimationFrame(() => open()))
        } else {
          open()
        }

        nativeEvent.stopPropagation()
      },
      updateRange ({ start, end }) {
        const events = []

        const min = new Date(`${start.date}T00:00:00`)
        const max = new Date(`${end.date}T23:59:59`)
        const days = (max.getTime() - min.getTime()) / 86400000
        const eventCount = this.rnd(days, days + 20)

        for (let i = 0; i < eventCount; i++) {
          const allDay = this.rnd(0, 3) === 0
          const firstTimestamp = this.rnd(min.getTime(), max.getTime())
          const first = new Date(firstTimestamp - (firstTimestamp % 900000))
          const secondTimestamp = this.rnd(2, allDay ? 288 : 8) * 900000
          const second = new Date(first.getTime() + secondTimestamp)

          events.push({
            name: this.names[this.rnd(0, this.names.length - 1)],
            start: first,
            end: second,
            color: this.colors[this.rnd(0, this.colors.length - 1)],
            timed: !allDay,
          })
        }

        this.events = events
        console.log(events)
      },
      rnd (a, b) {
        return Math.floor((b - a + 1) * Math.random()) + a
      },
      createTodo() {
            const todoItem = {
                content: this.newTodo,
                done: false,
                date: this.focus,
            }

            if(this.newTodo) {
                // 값이 있을때
                // console.log("createTodo!!!");
                // console.log(todoItem);
                this.todos.push(todoItem);
            } else {
                alert("내용 입력중")
                this.$refs.todoInput.focus();
            }
            this.newTodo = '';
      },
      doneTodo(todo) {
        todo.todoDone = !todo.todoDone;
      },
      deleteTodo(todo) {
        const todoItem = {
                todoNo: todo.todoNo,
                userId: this.user.userId,
                todoDate: this.focus,
            }
        this.$store.dispatch("removeTodo", todoItem);
      },
      setTodo() {
        const todoItem = {
                todoContent: this.newTodo,
                userId: this.user.userId,
                todoDate: this.focus,
            }

        if(this.newTodo) {
                // 값이 있을때
                // console.log("createTodo!!!");
                // console.log(todoItem);
                // this.todos.push(todoItem);
                this.$store.dispatch("writeTodo", todoItem);
            } else {
                alert("내용 입력중")
                this.$refs.todoInput.focus();
            }
        this.newTodo = '';
      },
    },
  }
</script>

<style>
  .click-done {
        cursor: pointer;
  }
  .is-done {
        text-decoration: line-through;
  }
</style>