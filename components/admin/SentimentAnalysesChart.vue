<template>
    <line-chart style="max-height: 300px" :chart-data="sentimentAnalysesChartData" :options="options"></line-chart>
</template>

<script>
  import LineChart from '@/components/charts/LineChart.js'

  export default {
    components: {
        LineChart
    },
    data () {
        return {
            options: {
                scales:{
                xAxes: [{
                        display: false //this will remove all the x-axis grid lines
                    }]
                },
                legend: {
                    display: false
                },
                responsive: true,
                maintainAspectRatio: false
            }
        }
    },
    props: {
        sentimentAnalysesChartData: Object
    }
  }
</script>
