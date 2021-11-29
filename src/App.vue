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
  </div>
</template>

<script>
import MyTable from "@/components/my-table";
import MyTableColumn from "@/components/my-table-column";
// import VirtualScrollWrapper from '@/components/virtual-scroll';

export default {
  name: "App",
  components: {
    MyTable,
    MyTableColumn,
    // VirtualScrollWrapper
  },
  data() {
    return {
      rows: [],
    };
  },
  async created() {
    const res = await fetch(`https://jsonplaceholder.typicode.com/comments`);
    this.rows = await res.json();
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
