<script>
import FilterDropdown from "./filter-dropdown";
import { orderBy } from "lodash/collection";
import DotsLoaderIcon from './dots-loader.svg';

export default {
  name: "my-table",
  props: {
    rows: {
      type: Array,
      required: true,
    },
    totalPages: {
      type: Number,
      default: 0
    },
    currentPage: {
      type: Number,
      default: 0
    },
    staticPaging: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      sortProp: "",
      sortDirection: "",
      filterProp: "",
      filterText: "",
    };
  },
  computed: {
    sortedRows() {
      let res;

      if (!this.sortProp) {
        res = this.rows;
      }

      res = orderBy(this.rows, [this.sortProp], [this.sortDirection]);

      if (this.filterText) {
        res = res.filter(
          (row) => row[this.filterProp].toLowerCase().search(this.filterText.toLowerCase()) > -1
        );
      }

      return res;
    },
  },
  methods: {
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
    },
    openFilterTooltip(prop = "") {
      this.filterProp = prop;
    },
    setFilterText(e) {
      this.filterText = e.target.value;
    },
    renderInfPager() {
      const directives = [
        {
          name: 'detect-viewport',
          value: {
            callback: this.$listeners.getPage
          }
        }
      ];

      const style = {
        background: `url("${DotsLoaderIcon}") no-repeat center`
      };

      return (
        <div {...{ class: this.$style.infPager, style, directives }} />
      );
    }
  },
  
  render(h) {
    const {
      sortProp,
      sortDirection,
      /* filterProp, */ filterText,
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
        h("span", {}, renderedTitle),
        h("font-awesome-icon", {
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
      ]);
    });

    const rows = this.sortedRows.map((row, index) => {
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
}
.my-table__thead-th {
  width: 30%;
  background-color: lightgrey;
}
.my-table__thead-th:first-child,
.my-table__thead-th:nth-child(2) {
  width: 5%;
}

.my-table__thead-th,
.my-table__tbody-td {
  padding: 10px 20px;
  text-align: left;
}
</style>
