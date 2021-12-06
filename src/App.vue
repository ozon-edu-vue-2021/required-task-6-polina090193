<template>
  <div id="app">
    <my-table
      v-if="dataIsLoaded"
      :currentPage="currentPage"
      :rows="rows"
      :totalPages="totalPages"
      :tableJson="tableJson"
      :infiniteScroll="infiniteScroll"

      @getPage="getPage"
    >
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
    <DotsLoaderIcon v-if="infiniteScroll && currentPage < totalPages && loading" v-view.once="infGetPage(currentPage + 1)" />
    <MyTablePaginator
      v-else-if="!infiniteScroll"
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
import DotsLoaderIcon from '@/components/dots-loader.svg';

export default {
  name: "App",
  components: {
    MyTable,
    MyTableColumn,
    MyTablePaginator,
    DotsLoaderIcon
  },
  data() {
    return {
      dataIsLoaded: false,
      tableJson: [],
      sortedRows: [],
      rows: [],
      currentPage: 1,
      infiniteScroll: true,
      pageSize: 10,
      totalPages: 100,
    };
  },
  provide() {
    return {
      getPage: this.getPage,
      infGetPage: this.infGetPage,
    };
  },
  async created() {
    const res = await fetch(`https://jsonplaceholder.typicode.com/comments`);
    this.tableJson = await res.json();

    this.dataIsLoaded = true;

    /* this.infiniteScroll && ( */this.blockingPromise = this.getPage(this.currentPage)/* ) */;
  },
  methods: {
    /* async */ getPage(number, content = this.sortedRows) {
      this.sortedRows = content;
      let rows = content.filter((row, index) => {
        let start = (number - 1) * this.pageSize;
        let end = number * this.pageSize;
        if (index >= start && index < end) return true;
      });
      this.currentPage = number;
      this.rows = rows;
    },
    async infGetPage(number = this.currentPage, content = this.sortedRows) {
      this.blockingPromise && (await this.blockingPromise);
      this.sortedRows = content;
      let end = number * this.pageSize;
      console.log('number = ', number);
      let rows = content.filter((row, index) => {
        if (index >= 1 && index < end) return true;
      });
      // console.log(rows);
      this.currentPage++
      this.rows = rows;
      this.totalPages = Math.ceil(content.length / this.pageSize);
      /* this.sortedRows = [...this.sortedRows, ...this.tableJson]; */
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
