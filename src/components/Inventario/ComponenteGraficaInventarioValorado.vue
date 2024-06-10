<template>
  <div>
    <section class="m-4">
      <h1 class="text-xl font-semibold mb-1 text-center ml-[5%]">Inventario de Productos</h1>
      <div class="border p-4 m-2">
        <apexchart width="100%" height="350%" type="bar" :options="chartOptions" :series="series"></apexchart>
      </div>
    </section>

    <section id="resumen" class="m-4">
      <h2 class="text-center font-bold text-lg">Tabla de productos</h2>

      <div class="m-2 grid grid-cols-12 items-end">
        
        <div class="col-span-3">
          <label for="filter_column">Ordenar por </label>
          <select 
          class="rounded border-slate-300"
          @change="cargarDatosGrafico()" name="filter_column" v-model="order_column" id="select-filter-column">
            <option value="existencias">Cantidad</option>
            <option value="lotes_disponibles">Lotes</option>
            <option value="valor_monetario">Valor($)</option>
          </select>
        </div>
        <div class="col-span-3">
          <label for="filter_column">Orden </label>
          <select 
          class="rounded border-slate-300"
          @change="cargarDatosGrafico()" name="orden" id="select-order" v-model="order_type">
            <option value="desc">Descendente</option>
            <option value="asc">Ascendente</option>
          </select>
        </div>
        <div class="col-span-3">
          <label for="filter_column">Mostrar </label>
          <select
          class="rounded border-slate-300"
           @change="cargarDatosGrafico()" name="orden" id="select-order" v-model="cant_rows">
            <option value="10">10</option>
            <option value="50">50</option>
            <option value="100">100</option>
          </select>
        </div>
      </div>
      <div class="mt-2">
        <table class="w-full m-auto rounded-lg shadow-lg mb-[2%]">
          <thead>
            <tr class="border-b-2 border-black-400 h-[40px] bg-slate-100">
              <th class=" font-bold">Producto</th>
              <th class=" font-bold">Cantidad</th>
              <th class=" font-bold">Cantidad de Lotes</th>
              <th class=" font-bold">Valor USD</th>
            </tr>
          </thead>
          <tbody class="bg-white">
            <tr class="border-b" v-for="producto in datos_productos">
              <td class="p-2 text-center">{{ producto.nombre_producto }}</td>
              <td class="p-2 text-center">{{ producto.existencias }}</td>
              <td class="p-2 text-center">{{ producto.lotes_disponibles }}</td>
              <td class="text-center">${{ producto.valor_monetario }}</td>
            </tr>
          </tbody>
          <tfoot>
            <tr>
              <td class="text-center font-bold p-2"></td>
              <td class="text-center font-bold p-2"></td>
              <td class="text-center font-bold p-2"></td>
            </tr>
          </tfoot>
        </table>
      </div>
    </section>
  </div>
</template>
<script>
import axios from 'axios';
export default {

  data() {
    return {
      order_column:'existencias',
      order_type:'desc',
      cant_rows:10,
      datos_productos: [],
      series: [
        {
          name: 'Valor USD($)',
          data: [

          ],
          isPrice: true
        },
        {
          name: "Existencias",
          data: [0],
          isPrice: false
        }
      ],
      chartOptions: {
        chart: {
          type: "bar",
          height: '250px',
          toolbar: {
            show: true
          },
        },
        plotOptions: {
          bar: {
            columnWidth: '70%',
            horizontal: false,
            dataLabels: {
              position: "top"
            },
          }
        },
        dataLabels: {
          position: top,
          enabled: true,
          offsetX: -6,
          style: {
            fontSize: "10px",
            colors: [
              "#000"
            ]
          },
          formatter: function (val, opts) {
            // Use opts.seriesIndex to get the index of the series
            var seriesIndex = opts.seriesIndex;
            // Check the custom property to determine if it is a price series
            if (opts.w.config.series[seriesIndex].isPrice) {
              return '$' + val;
            } else {
              return val;
            }
          },
        },
        stroke: {
          show: true,
          width: 1,
          colors: [
            "#fff"
          ]
        },
        tooltip: {
          shared: true,
          intersect: false
        },
        xaxis: {
          title: {
            text: 'Productos'
          },
          categories: [

          ]
        },
        yaxis: {
          title: {
            text: 'Cantidades'
          }
        },
        legend: {
          position: 'top',

        },
      }
    }
  },
  mounted() {
    this.cargarDatosGrafico();
  },
  methods: {
    cargarDatosGrafico() {
      const params = {
        "order_column" : this.order_column,
        'order_type' : this.order_type,
        'cant_rows' : this.cant_rows
      }
      console.log(params)
      axios.get("/api/datos_inventario_valorado",{params})
        .then(
          (response) => {
            this.clearData();
            this.datos_productos = response.data.datos_inventario;
            console.log(response.data.categories);
            this.chartOptions.xaxis.categories.splice(0, this.chartOptions.xaxis.categories.length);
            this.series[0].data = response.data.data;
            this.series[1].data = response.data.existencias
            response.data.categories.forEach((element) => {
              this.chartOptions.xaxis.categories.push(element);
            });
          }
        )
        .catch(
          (response) => {
            console.log(response);
            this.clearData();
          }
        );
    },
    clearData(){
      //this.chartOptions.xaxis.categories = []
      this.datos_productos = [];
    }
  }
}
</script>
