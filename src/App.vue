<template>
  <div id="app">
    <my-table :rows="rows">
      <my-table-column prop="id" title="ID">
        <template slot-scope="{ row }">
          <b>{{ row.id }}</b>
        </template>
      </my-table-column>

      <my-table-column prop="postId" title="Post ID">
        <template slot-scope="{ row }">
          <b>{{ row.postId }}</b>
        </template>
      </my-table-column>

      <my-table-column prop="name" title="Name" />

      <my-table-column prop="email" title="Email">
        <template #title>
          <b>Email</b>
        </template>

        <template #body="{ row }">
          <a :href="`mailto:${row.email}`">{{ row.email }}</a>
        </template>
      </my-table-column>

      <my-table-column prop="body" title="Text">
        <template slot-scope="{ row }">
          {{ row.body }}
        </template>
      </my-table-column>
    </my-table>
    <MyTablePaginator
      v-if="paging === 'static'"
      :totalPages="totalPages"
      :currentPage="currentPage"
      @getPage="getPage"
    />
  </div>
</template>

<script>
import MyTable from "@/components/my-table";
import MyTableColumn from "@/components/my-table-column";
import MyTablePaginator from "@/components/my-table-paginator";

export default {
  name: "App",
  components: {
    MyTable,
    MyTableColumn,
    MyTablePaginator,
  },
  data() {
    return {
      tableJson: [],
      rows: [],
      currentPage: 1,
      paging: "static", // 'infinite', 'virtual'
      pageSize: 10,
      totalPages: 1,
    };
  },
  async created() {
    const res = await fetch(`https://jsonplaceholder.typicode.com/comments`); // ?postId=${number}
    this.tableJson = await res.json();
    this.totalPages = Math.floor(this.tableJson.length / this.pageSize);
    this.blockingPromise = this.getPage(this.currentPage);
  },
  methods: {
    async getPage(number) {
      this.rows = this.tableJson.filter((row, index) => {
        let start = (number - 1) * this.pageSize;
        let end = number * this.pageSize;
        if (index >= start && index < end) return true;
      });
      this.currentPage = number;
    },
    async infGetPage() {
      this.blockingPromise && (await this.blockingPromise);
      this.rows = [...this.rows, ...this.tableJson];
      this.currentPage++;
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
