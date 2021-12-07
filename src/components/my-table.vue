<script>
import FilterDropdown from "./filter-dropdown";
import { orderBy } from "lodash/collection";
import DotsLoaderIcon from "./dots-loader.svg";

export default {
  name: "my-table",
  props: {
    tableJson: {
      type: Array,
      required: true,
    },
    rows: {
      type: Array,
      required: true,
    },
    totalPages: {
      type: Number,
      default: 0,
    },
    currentPage: {
      type: Number,
      default: 0,
    },
    infiniteScroll: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      sortProp: "",
      sortDirection: "",
      filterProp: "",
      filterText: "",
    };
  },
  inject: ['getPage', 'infGetPage'],
  methods: {
    sortRows() {
      let res;

      if (!this.sortProp) {
        res = this.tableJson;
      }

      res = orderBy(this.tableJson, [this.sortProp], [this.sortDirection]);

      if (this.filterText) {
        res = res.filter(
          (row) =>
            row[this.filterProp]
              .toString().toLowerCase()
              .search(this.filterText.toString().toLowerCase()) > -1
        );
      }

      this.infiniteScroll ? this.infGetPage(1, res) : this.getPage(1, res)
    },
    renderColumns(h, row, columnsOptions) {
      return columnsOptions.map((column) => {
        return h("td", { class: "my-table__tbody-td" }, [
          column.scopedSlot.body
            ? column.scopedSlot.body({ row, items: this.rows })
            : row[column.prop],
        ]);
      });
    },
    toggleSort(prop) {
      this.sortProp = prop;
      this.sortDirection =
        this.sortDirection === "desc" || !this.sortDirection ? "asc" : "desc";
        this.sortRows()
    },
    openFilterTooltip(prop = "") {
      this.filterProp = prop;
      this.filterText = '';
    },
    setFilterText(e) {
      this.filterText = e.target.value;
      this.sortRows()
    },
    renderInfPager() {
      const directives = [
        {
          name: "detect-viewport",
          value: {
            callback: this.$listeners.getPage,
          },
        },
      ];

      const style = {
        background: `url("${DotsLoaderIcon}") no-repeat center`,
      };

      return <div {...{ class: this.$style.infPager, style, directives }} />;
    },
  },

  mounted() {
    this.sortRows()
  },

  render(h) {
    const {
      sortProp,
      sortDirection,
      filterText,
      setFilterText,
    } = this;

    const columnsOptions = this.$slots.default
      .filter((item) => {
        return (
          item.componentOptions &&
          item.componentOptions.tag === "my-table-column"
        );
      })
      .map((column) => {
        return Object.assign({}, column.componentOptions.propsData, {
          scopedSlot: column.data.scopedSlots || {},
        });
      });

    const columnsHead = columnsOptions.map((column) => {
      const renderedTitle = column.scopedSlot.title
        ? column.scopedSlot.title()
        : column.title;
      let sortIcon = "sort";

      if (sortProp === column.prop) {
        sortIcon =
          sortDirection === "asc" ? "sort-amount-down" : "sort-amount-up";
      }

      return h("th", { class: "my-table__thead-th", key: column.prop }, [
        h("div", { class: "my-table__thead-wrap" }, [
          h("span", { class: "my-table__thead-name" }, renderedTitle),
          h("font-awesome-icon", {
            class: "my-table__thead-icon",
            props: {
              icon: sortIcon,
            },
            on: { click: () => this.toggleSort(column.prop) },
          }),
          h(FilterDropdown, {
            props: {
              columnProp: column.prop,
              filterText: filterText,
            },
            on: {
              openFilterTooltip: () => {
                this.openFilterTooltip(column.prop);
              },
              setFilterText: setFilterText,
            },
          }),
        ]),
      ]);
    });

    const rows = this.rows.map((row, index) => {
      return h("tr", { key: index }, [
        this.renderColumns(h, row, columnsOptions),
      ]);
    });

    return h("table", { class: "my-table" }, [
      h("thead", { class: "my-table__thead" }, [h("tr", [columnsHead])]),
      h("tbody", { class: "my-table__tbody" }, [rows]),
    ]);
  },
};
</script>
<style>
.my-table {
  width: 80%;
  margin: auto;
  margin-top: 10px;
}
.my-table__thead-th {
  width: 20%;
  background-color: lightgrey;
}
.my-table__thead-th:first-child,
.my-table__thead-th:nth-child(2) {
  width: 10%;
}

.my-table__thead-th,
.my-table__tbody-td {
  padding: 5px 10px;
  text-align: left;
}
.my-table__thead-wrap {
  display: flex;
}
.my-table__thead-icon {
  white-space: nowrap;
  margin-left: auto;
}
.filter-dropdown_filterIcon_1emoy {
  margin-left: 10px;
}
</style>
