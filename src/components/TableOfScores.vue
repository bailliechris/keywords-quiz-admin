<template>
    <div class="container">
        <b-table :data="scorelist" :columns="columns"></b-table>
    </div>
</template>

<script>
export default {
  name: 'TableOfScores',
  props: ["scorelist"],
  data(){
      return{
          columns: [
                    {
                        field: 'topic',
                        label: 'Topic',
                        sortable: true
                    },
                    {
                        field: 'name',
                        label: 'Name',
                        sortable: true
                    },
                    {
                        field: 'score',
                        label: 'Score',
                        sortable: true
                    },
                    {
                        field: 'max',
                        label: 'Max',
                        sortable: true
                    },
                    {
                        field: 'time',
                        label: 'Time',
                        sortable: true
                    }                   
                ]
      }
  }
}
</script>