<template>
  <section>
    <h3 class="title is-3">{{ $t('component.admin.organizations', 'Organizations') }}</h3>
    <search-input id="organization-search"
                  :value="searchValue"
                  @search="searchOnInput" />
    <no-ssr>
      <v2-table :data="organizations.data"
                :stripe="true"
                :loading="isLoading"
                :total="organizations.total"
                :shown-pagination="totalPages > 1"
                :pagination-info="paginationInfo"
                @page-change="handlePageChange">
        <v2-table-column label="Name" prop="name" align="left" width="220">
          <template slot-scope="row">
            <hc-button type="link" target="_blank" :to="{name: 'organizations-slug', params: {slug: row.slug}}">
              {{ row.name }}
            </hc-button>
          </template>
        </v2-table-column>
        <v2-table-column label="Reviewd" prop="reviewer" align="center">
          <template slot-scope="row">
            <hc-button v-if="row.reviewedBy" :title="row.reviewer.name" type="link" target="_blank" :to="{name: 'profile-slug', params: {slug: row.reviewer.slug}}">
              <i v-show="row.reviewedBy" class="fa fa-check"></i>
            </hc-button>
          </template>
        </v2-table-column>
        <v2-table-column label="Created" prop="createdAt" align="left">
          <template slot-scope="row">
            <hc-relative-date-time :dateTime="row.createdAt" />
          </template>
        </v2-table-column>
        <v2-table-column label="" prop="language" align="center" width="42">
          <template slot-scope="row">
            <template v-if="row.language">
              <img width="16" :src="`/assets/svg/flags/${row.language}.svg`" />
            </template>
            <template v-else>
              -
            </template>
          </template>
        </v2-table-column>
      </v2-table>
    </no-ssr>
  </section>
</template>

<script>
  import moment from 'moment'
  import parse from 'csv-parse/lib/sync'
  import { isEmpty, each, map, keyBy } from 'lodash'
  import SearchInput from '~/components/Search/SearchInput.vue'

  let itemLimit = 10

  export default {
    middleware: 'admin',
    layout: 'admin',
    components: {
      SearchInput
    },
    data () {
      return {
        itemLimit: itemLimit,
        isLoading: true,
        currentPage: 1,
        paginationInfo: {
          text: '', // this.paginationText,
          nextPageText: '>',
          prevPageText: '<'
        },
        searchValue: ''
      }
    },
    async asyncData ({app}) {
      const limit = itemLimit
      const organizations = await app.$api.service('organizations').find({
        query: {
          $limit: limit,
          $sort: {
            createdAt: -1
          }
        }
      })
      return {
        organizations: organizations,
        isLoading: false,
        itemLimit: limit
      }
    },
    filters: {
      formatDate (val) {
        return moment(val).format('d.m.Y')
      }
    },
    computed: {
      baseURL () {
        return location.origin
      },
      totalPages () {
        return this.organizations.total / this.itemLimit
      },
      paginationText () {
        return `<span>Total of <strong>${this.user ? this.organizations.total : 0}</strong>, <strong>${this.itemLimit}</strong> per page</span>`
      }
    },
    methods: {
      searchOnInput (value) {
        this.searchValue = value
        this.handlePageChange(1)
      },
      async handlePageChange (page) {
        this.currentPage = page
        this.isLoading = true
        const start = (page - 1) * this.itemLimit

        const query = {
          $limit: this.itemLimit,
          $sort: {
            createdAt: -1
          },
          $skip: start
        }
        if (!isEmpty(this.searchValue)) {
          query.name = {
            $search: this.searchValue
          }
        }
        this.organizations = await this.$api.service('organizations').find({query})
        this.isLoading = false
      }
    },
    head () {
      return {
        title: 'Organisationen verwalten'
      }
    }
  }
</script>

<style scoped lang="scss">
  @import 'assets/styles/utilities';

  .cell-name {
    font-weight: bold;
  }

  a[disabled] {
    pointer-events: none;
  }

  .link {
    white-space: nowrap;
    cursor: pointer;
    color: $primary;
  }

  .fa-check-circle,
  .fa-check {
    color: $primary;
  }
</style>
