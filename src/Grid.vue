<!--
Generic table view based on bootstrap styles
Features:
  * Filtering on all keys
  * Pagination
  * Sorting and reverse sorting on any key
  * Optional custom renderers for individual columns
-->
<template>
  <div>
  <table class="table table-striped table-bordered table-hover">
    <thead>
      <tr>
        <th v-for="key in columns"
          @click="sortBy(key.path)"
          :class="[key.addClass, { act: sortKey == key.path}]">
          {{ key.title }}
          <span class="arrow" :class="sortOrders[key.path] > 0 ? 'asc' : 'dsc'">
          </span>
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="entry in limitedData" @click="rowEvent(entry)" class="grid-row">
        <td v-for="key in columns" :class="[key.addClass]"  class="grid-cell">
          <template v-if="key.render">
            {{ key.render(entry[key.path]) }}
          </template>
          <template v-else>
            {{entry[key.path] }}
          </template>
        </td>
      </tr>
      <tr v-if="isLoading">
        <td :colspan="columns.length" class="text-center noData">
          <span class="glyphicon glyphicon-repeat normal-right-spinner"></span>
        </td>
      </tr>
      <tr v-if="!isLoading && !data.length">
        <td :colspan="columns.length" class="text-center noData">
          {{emptyTableText}}
        </td>
      </tr>
    </tbody>
  </table>
    <div class="row" v-if="noPages.length > 1">
      <div class="text-center">
        <ul class="pagination">
          <li :class="{disabled: (currentPage === 0)}"><a href="#" @click="previous">Previous</a></li>
          <template>
            <li v-for="page in noPages" :class="{active: page === currentPage}">
              <a href="#" @click="(e) => goToPage(page, e)">{{page + 1}}</a>
            </li>
          </template>
          <li :class="{disabled: (currentPage + 1 === noPages.length)}"><a href="#" @click="next">Next</a></li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    props: {
      data: Array, // Unfiltered table data
      columns: Array, // List of columns ['title', 'description']
      filterKey: String, // String to filter on ''
      customEmptyTableText: '', // optional text override for empty table
      rowsPerPage: Number, // Number of rows per page of results
      isLoading: false
    },
    data: function () {
      var sortOrders = {}
      this.columns.forEach(function (key) {
        sortOrders[key.path] = 1
      })
      return {
        sortKey: '',
        sortOrders: sortOrders,
        currentPage: 0
      }
    },
    watch: {
      'filterKey': function (a, b) { // Reset to page 0 when search changes
        this.currentPage = 0
      }
    },
    computed: {
      noPages: function () { // No. pages as an array. Used for rendering paginator
        let pages = Math.ceil(this.filteredData.length / this.rowsPerPage)
        return [...Array(pages).keys()]
      },
      emptyTableText: function () { // Default text shown when there's no data in the table
        if (this.customEmptyTableText) {
          return this.customEmptyTableText
        } else {
          return 'No data to show'
        }
      },
      filteredData: function () { // Data filtered through the local search
        var sortKey = this.sortKey
        var filterKey = this.filterKey && this.filterKey.toLowerCase()
        var order = this.sortOrders[sortKey] || 1
        var data = this.data
        if (filterKey) {
          data = data.filter(function (row) {
            return Object.keys(row).some(function (key) {
              return String(row[key]).toLowerCase().indexOf(filterKey) > -1
            })
          })
        }
        if (sortKey) { // If sort specified, sort
          data = data.slice().sort(function (a, b) {
            a = a[sortKey]
            b = b[sortKey]
            return (a === b ? 0 : a > b ? 1 : -1) * order
          })
        }
        return data
      },
      limitedData: function () { // Data limited to a page
        return this.filteredData.slice(
          this.currentPage * this.rowsPerPage,
          this.rowsPerPage * (this.currentPage + 1)
        )
      }
    },
    methods: {
      sortBy: function (key) { // header row click handler. Sort by column
        this.sortKey = key
        this.sortOrders[key] = this.sortOrders[key] * -1
      },
      rowEvent: function (entry) { // Event broadcase from table with clicked element
        this.$emit('clickRow', entry)
      },
      previous: function (e) { // Previous page click handler
        e.preventDefault()
        if (this.currentPage !== 0) {
          --this.currentPage
        }
      },
      next: function (e) { // Next page click handler
        e.preventDefault()
        if (this.currentPage + 1 !== this.noPages.length) {
          ++this.currentPage
        }
      },
      goToPage: function (page, e) { // Go to a specific page click handler
        e.preventDefault()
        this.currentPage = page
      }
    }
  }
</script>

<style scoped>
  .grid-row {
    cursor: pointer;
  }

  th {
    cursor: pointer;
  }

  th, td {
    padding: 10px 20px;
  }

  th.act {
    color: #fff;
  }

  th.act .arrow {
    opacity: 1;
  }

  .arrow {
    display: inline-block;
    vertical-align: middle;
    width: 0;
    height: 0;
    margin-left: 5px;
    opacity: 0.66;
  }

  .arrow.asc {
    border-left: 4px solid transparent;
    border-right: 4px solid transparent;
    border-bottom: 4px solid #fff;
  }

  .arrow.dsc {
    border-left: 4px solid transparent;
    border-right: 4px solid transparent;
    border-top: 4px solid #fff;
  }

  .glyphicon.normal-right-spinner {
    -webkit-animation: glyphicon-spin-r 2s infinite linear;
    animation: glyphicon-spin-r 2s infinite linear;
    font-size: 2em;
  }

  @-webkit-keyframes glyphicon-spin-r {
    0% {
      -webkit-transform: rotate(0deg);
      transform: rotate(0deg);
    }

    100% {
      -webkit-transform: rotate(359deg);
      transform: rotate(359deg);
    }
  }

  @keyframes glyphicon-spin-r {
    0% {
      -webkit-transform: rotate(0deg);
      transform: rotate(0deg);
    }

    100% {
      -webkit-transform: rotate(359deg);
      transform: rotate(359deg);
    }
  }
</style>
