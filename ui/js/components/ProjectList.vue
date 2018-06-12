<template>
  <section>
    <component-project-create v-on:project-created="projectCreated"></component-project-create>
    <hr />
    <section>
      <h2 class="subtitle">Projects</h2>
      <b-field grouped>
      </b-field>
      <b-table 
        :data="data" 
        :loading="loading"

        :default-sort-direction="defaultSortDirection"
        :default-sort="[sortField, sortOrder]"

        paginated
        backend-pagination
        :total="total"
        :per-page="perPage"
        @page-change="onPageChange">

        <template slot-scope="props">
          <b-table-column field="id" label="ID" width="100" sortable>
            {{ props.row.id }}
          </b-table-column>

          <b-table-column field="name" label="Name" width="200" sortable>
            {{ props.row.name }}
          </b-table-column>

          <b-table-column field="description" label="Description">
            {{ props.row.description | truncate(80) }}
          </b-table-column>

          <b-table-column width="100">
            <router-link :to="{ name: 'project', params: { id: props.row.id } }" class="button block" @click="detailsClicked(props.row.id, $event)">Details</router-link>
          </b-table-column>
        </template>
      </b-table>
    </section>
  </section>
</template>

<script>
import axios from 'axios'
import ProjectCreate from './ProjectCreate.vue'

export default {
  data() {
    return {
      data: [],
      total: 100,
      loading: false,
      sortField: 'id',
      sortOrder: 'desc',
      defaultSortDirection: 'desc',
      page: 1,
      perPage: 20
    }
  },
  methods: {
    async loadAsyncData() {
      const page = this.page - 1 || 0
      const params = `page=${page}`

      this.loading = true
      try {
        const { data } = await axios.get(
          `http://localhost:3000/api/projects?${params}`
        )

        this.data = data
        this.loading = false
      } catch (error) {
        this.data = []
        this.total = 0
        this.loading = false
        throw error
      }
    },

    onPageChange(page) {
      this.page = page
      this.loadAsyncData()
    },

    onSort(field, order) {
      this.sortField = field
      this.sortOrder = order
      this.loadAsyncData()
    },

    detailsClicked(id, ev) {
      console.log(id)
    },

    projectCreated(p) {
      this.loadAsyncData()
    }
  },
  filters: {
    /**
     * Filter to truncate string, accepts a length parameter
     */
    truncate(value, length) {
      return value.length > length ? value.substr(0, length) + '...' : value
    }
  },
  mounted() {
    this.loadAsyncData()
  },
  components: {
    'component-project-create': ProjectCreate
  }
}
</script>