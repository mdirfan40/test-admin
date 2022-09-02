<script>
import EditIcon from "./components/icons/EditIcon.vue";
import DeleteIcon from "./components/icons/DeleteIcon.vue";
export default {
  data() {
    return {
      pageSize: 10,
      isEdit: false,
      edit: {},
      page: 0,
      selectedIds: [],
      search: "",
      selectAllCheckbox: false,
      db: [],
      buffer: [],
      isLoading: false
    };
  },
  watch: {
    search(val) {
      if (!val) {
        this.buffer = this.db;
        return;
      }
      this.buffer = this.db.filter((item) => {
        const regex = new RegExp(val, "i");
        if (regex.test(item.name)) {
          return true;
        } else if (regex.test(item.role)) {
          return true;
        } else if (regex.test(item.email)) {
          return true;
        } else {
          return false;
        }
      });
    }
  },
  computed: {
    diplayPage() {
      return this.paginate();
    },
    pageCount() {
      return Math.ceil(this.buffer.length / this.pageSize);
    },
    isLast() {
      return this.pageCount <= this.page + 1;
    },
    from() {
      return this.page * this.pageSize;
    },
    to() {
      return this.from + this.pageSize;
    },
    hasSelected() {
      return (id) => this.selectedIds.includes(id);
    }
  },
  methods: {
    paginate() {
      return this.buffer.slice(this.from, this.to);
    },
    next() {
      this.page++;
    },
    prev() {
      this.page--;
    },
    changePage(page) {
      this.page = page;
    },
    deleteItem(id) {
      this.buffer = this.buffer.filter((item) => item.id != id);
      this.db = this.db.filter((item) => item.id != id);
    },
    deleteSingle(id) {
      const confirm = window.confirm("Do you really want to delete");

      if (confirm) {
        this.deleteItem(id);
      }
    },
    deleteSelected() {
      const confirm = window.confirm("Do you really want to delete");
      if (confirm) {
        this.selectedIds.forEach((id) => this.deleteItem(id));
        this.selectedIds = [];
        this.selectAllCheckbox = false;
      }
    },
    selectAll(e) {
      if (e.target.checked) {
        for (let i = this.from; i < this.to; i++) {
          this.selectedIds.push(this.buffer[i].id);
        }
      } else {
        this.selectedIds = [];
      }
    },
    updateDB(data) {
      // update orginal db in linear time
      const index = this.db.findIndex((x) => x.id === data.id);
      this.db[index] = { ...data };
    },
    onEdit(i) {
      let editObj = this.buffer[i];
      this.edit = { ...editObj, index: i };
      this.isEdit = true;
    },
    save() {
      this.buffer[this.edit.index] = { ...this.edit };
      this.isEdit = false;
      this.updateDB(this.edit);
    }
  },
  async mounted() {
    this.isLoading = true;
    try {
      const res = await fetch(
        "https://geektrust.s3-ap-southeast-1.amazonaws.com/adminui-problem/members.json"
      );
      this.db = await res.json();
      this.buffer = this.db;
    } catch (e) {
      console.log(e);
    } finally {
      this.isLoading = false;
    }
  },
  components: { EditIcon, DeleteIcon }
};
</script>

<template>
  <main>
    <div
      v-if="isEdit"
      style="
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        padding: 15px;
        background-color: rgba(0, 0, 0, 0.3);
        z-index: 100;
      "
    >
      <div
        style="
          background-color: white;
          max-width: 600px;
          border-radius: 5px;
          padding: 15px;
          margin: 5% auto 0 auto;
        "
      >
        <div>
          <h2>Edit</h2>
        </div>

        <div class="input">
          <input v-model="edit.name" type="text" placeholder="name" />
        </div>
        <div class="input">
          <input v-model="edit.email" type="text" placeholder="Email" />
        </div>
        <div class="input">
          <input v-model="edit.role" type="text" placeholder="Email" />
        </div>

        <div style="margin-top: 2.3rem">
          <button class="btn-primary" @click="save">Save</button>
          <button class="btn-danger" @click="isEdit = false">Cancel</button>
        </div>
      </div>
    </div>

    <div class="container">
      <div class="card">
        <h1>Admin</h1>
        <div class="input">
          <input v-model="search" type="text" placeholder="Search" />
        </div>
        <table>
          <tr>
            <th>
              <input
                @change="selectAll"
                type="checkbox"
                v-model="selectAllCheckbox"
              />
            </th>
            <th>Name</th>
            <th>Email</th>
            <th>Role</th>
            <th>Action</th>
          </tr>
          <tr v-show="isLoading">
            Loading...
          </tr>
          <tr
            :class="{ 'active-row': hasSelected(row.id) }"
            v-for="(row, i) in diplayPage"
            :key="row.id"
            v-show="row.show == null || row.show == true"
          >
            <td>
              <input :value="row.id" type="checkbox" v-model="selectedIds" />
            </td>
            <td>{{ row.name }}</td>
            <td>{{ row.email }}</td>
            <td>{{ row.role }}</td>
            <td class="action">
              <button @click="onEdit(i)" class="icon-button">
                <EditIcon></EditIcon>
              </button>
              <button @click="deleteSingle(row.id)" class="icon-button">
                <DeleteIcon></DeleteIcon>
              </button>
            </td>
          </tr>
        </table>

        <div class="footer">
          <div class="footer-action">
            <div>
              <transition>
                <button
                  @click="deleteSelected"
                  v-if="selectedIds.length > 0"
                  class="btn-danger"
                >
                  Delete ({{ this.selectedIds.length }})
                </button>
              </transition>
            </div>
            <div v-if="buffer.length > 0" class="pagination">
              <button :disabled="page === 0" @click="prev" class="prev">
                Prev
              </button>
              <button
                v-for="i in pageCount"
                @click="changePage(i - 1)"
                :key="i"
                :class="{ active: i - 1 == page }"
              >
                {{ i }}
              </button>
              <button :disabled="isLast" @click="next" class="next">
                Next
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
table {
  border-collapse: collapse;
  width: 100%;
}

td,
th {
  text-align: left;
  padding: 0.5rem;
}

th {
  font-weight: bold;
}

tr {
  border-bottom: solid 1px rgb(225, 225, 225);
  transition: all 0.5s ease;
}

tr:last-child {
  border-bottom: solid 1px transparent;
}
.input {
  margin-bottom: 0.8rem;
}

.input input {
  width: 100%;
  background-color: rgb(249, 249, 249);
  outline: none;
  border: solid 1px rgb(225, 225, 225);
  padding: 8px 10px;
  border-radius: 5px;
  font-size: 1rem;
}

input:focus {
  border: solid 1px rgb(155, 155, 155);
}
.icon-button {
  background: transparent;
  cursor: pointer;
  border: none;
  padding: auto;
  border-right: solid 1px rgb(225, 225, 225);
}

.icon-button:last-child {
  border-right: transparent;
}

.icon-button:hover {
  background-color: rgb(249, 249, 249);
}

.footer {
  margin-top: 3rem;
}

.footer-action {
  display: flex;
  justify-content: space-between;
}

.pagination button {
  background-color: rgb(249, 249, 249);
  font-size: 0.9rem;
  cursor: pointer;
  padding: 7px 10px;
  border: solid 1px rgb(220, 220, 220);
}

.pagination button:active {
  background-color: rgb(225, 225, 225);
}

.pagination .prev {
  border-top-left-radius: 100px;
  border-bottom-left-radius: 100px;
}

.pagination .active {
  background-color: blue;
  color: white;
  border: none;
}
.pagination .next {
  border-top-right-radius: 100px;
  border-bottom-right-radius: 100px;
}

.active-row {
  background-color: #deedf6;
}
</style>
