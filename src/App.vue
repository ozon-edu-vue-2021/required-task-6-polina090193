<template>
  <div id="app">
    <button @click="togglePaging">Toggle pagination paging/scroll</button>
    <my-table
      v-if="dataIsLoaded"
      :currentPage="currentPage"
      :rows="rows"
      :totalPages="totalPages"
      :tableJson="tableJson"
      :infiniteScroll="infiniteScroll"
      v-view="checkScrollPercent"

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
    <DotsLoaderIcon v-if="infiniteScroll && loading" />
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
import DotsLoaderIcon from "@/components/dots-loader.svg";

export default {
  name: "App",
  components: {
    MyTable,
    MyTableColumn,
    MyTablePaginator,
    DotsLoaderIcon,
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
      loading: true,
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

    // this.infiniteScroll && (this.blockingPromise = this.infGetPage(this.currentPage));
  },
  methods: {
    togglePaging() {
      this.infiniteScroll = !this.infiniteScroll;
      this.infGetPage(1);
    },
    getPage(number, content = this.sortedRows) {
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
      (this.loading = true),
        // this.blockingPromise && (await this.blockingPromise);
        (this.sortedRows = content);
      let rows = content.filter((row, index) => {
        let end = number * this.pageSize;
        if (index >= 0 && index < end) return true;
      });
      this.rows = rows;
      this.currentPage = number;
      this.totalPages = Math.ceil(content.length / this.pageSize);
      this.loading = false;

      console.log(this.$el.clientHeight);
      console.log(window.innerHeight);
      this.$nextTick(function() {
        if (this.$el.clientHeight < window.innerHeight && this.currentPage < this.totalPages) {
          this.infGetPage(this.currentPage + 1);
        }
      })
    },
    checkScrollPercent(e) {
      if (
        this.infiniteScroll &&
        this.currentPage < this.totalPages &&
        (e.scrollPercent === 1 || e.percentInView === 1)
      ) {
        this.infGetPage(this.currentPage + 1);
      }
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
