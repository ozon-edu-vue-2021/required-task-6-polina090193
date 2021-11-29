<script>
// import { get } from 'lodash'

export default {
  name: 'my-table',
  props: {
    rows: {
      type: Array,
      required: true
    }
  },
  methods: {
    renderColumns(h, row, columnsOptions) {
      return columnsOptions.map((column) => {
        return h('td', { class: 'my-table__tbody-td' }, [
          column.scopedSlot.body ? column.scopedSlot.body({row, items: this.rows}) : row[column.prop]
        ])
      })
    }
  },
  render(h) {
    const columnsOptions = this.$slots.default.filter(item => {
      return (item.componentOptions && item.componentOptions.tag === 'my-table-column')
    }).map(column => {
      return Object.assign({}, column.componentOptions.propsData, {
          scopedSlot: column.data.scopedSlots || {}
        }
      )
    })

    const columnsHead = columnsOptions.map((column, index) => {
      return h('th', { class: 'my-table__thead-th', key: index}, column.title)
    })

    const rows = this.rows.map((row, index) => {
      return h('tr', { key: index }, [ this.renderColumns(h, row, columnsOptions) ])
    })

    return h('table', { class: 'my-table' }, [
      h('thead', { class: 'my-table__thead' }, [
        h('tr', [ columnsHead ])
      ]),
      h('tbody', { class: 'my-table__tbody' }, [ rows ])
    ])
  }
};
</script>