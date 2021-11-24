<template>
  <el-table
    id="priceTable"
    :data="
      tableData.filter(
        (data) =>
          !search || data.site.toLowerCase().includes(search.toLowerCase())
      )
    "
    style="width: 75%; margin: 20px auto;"
    :default-sort="{ prop: 'line', order: 'ascending' }"
    v-model="price"
  >
    <el-table-column
      prop="line"
      label="线路"
      sortable
      width="300px"
      :sort-method="sortLine"
      align="center"
    >
    </el-table-column>
    <el-table-column prop="site" label="站点" width="300px" align="center">
    </el-table-column>
    <el-table-column prop="price" label="票价" sortable align="center">
    </el-table-column>
    <el-table-column align="right" width="300px">
      <!-- eslint-disable-next-line -->
      <template slot="header" slot-scope="scope">
        <el-input v-model="search" size="mini" placeholder="目的地检索" />
      </template>
    </el-table-column>
  </el-table>
</template>

<script>
export default {
  name: "PriceTable",
  data() {
    return {
      search: "",
      tableData: [],
    };
  },
  props: ["lineSite", "price"],
  watch: {
    price: function() {
      let lineSite = this.lineSite;
      let price = this.price;
      if (lineSite == null) return;

      let tmp = new Array();
      lineSite.forEach(function(value, key) {
        let line = key;
        let site = value;
        for (let i = 0; i < site.length; i++) {
          let obj = {};
          obj.line = line;
          obj.site = site[i];
          obj.price = price.get(obj.site);
          tmp.push(obj);
        }
      });
      this.$data.tableData = tmp;
    },
  },
  methods: {
    sortLine(a, b) {
      let a2 = parseInt(a);
      let b2 = parseInt(b);
      if (a2 > b2) return true;
      else return false;
    },
  },
};
</script>

<style>
#priceTable {
  /* width: 60%; */
  margin: 20px;
}
</style>
