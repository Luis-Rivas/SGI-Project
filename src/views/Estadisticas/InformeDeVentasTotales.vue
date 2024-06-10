<template>
  <main class="relative">
    <NavBar />

    <!-- Encabezado -->
    <div>
      <div class="flex bg-white mx-auto p-5 shadow-md justify-between">
        <h1 class="font-bold text-blue-700 text-xl">Informe de ventas Totales</h1>
      </div>
      <div class="flex justify-start items-center mt-4 ml-4">
        <a href="#" @click="$router.go(-1)" class="text-sm text-black font-medium flex items-center">
          <img src="../../assets/icons/arrow.svg" alt="Regresar" class="h-6 w-6 mr-1" /> Regresar
        </a>
      </div>
    </div>

    <div class="grid grid-cols-10 gap-4 w-[90%] m-auto">
      <div class="col-span-8 flex align-cente">
        <div class="grid grid-cols-6 gap-4 w-[45%] ml-[10%] mt-[4%]">
          <div v-for="mes in opcionesFiltroMeses" :key="mes.mes">
            <input type="checkbox" class="inline-block rounded-[3px]" :id="mes.mes" v-model="mes.estaActivo" />
            <label :for="mes.mes" class="inline-block ml-[10px]">{{ mes.mes }}</label>
          </div>
        </div>
        <div class="cols-pan-2 flex align-middle justify-center items-center mx-4">
          <button @click="selectAllMonth()"
            class="h-fit btn bg-blue-700 rounded-md text-white p-2 hover:bg-blue-900">Seleccionar
            todos</button>
        </div>
      </div>
      <div class="col-span-2">
        <div class="w-[100%] ">
          <div class="mt-[5%] ml-[7%]">
            <button type="button"
              class="text-white bg-indigo-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800"
              @click="obtenerDatosFiltrados">
              Aplicar
            </button>
          </div>
          <div class="p-4 mr-[10%]">
            <select name="filtroAnio" id="filtroAnio" class="border-slate-300 rounded text-slate-400 bg-slate-50"
              v-model="fechaFiltro">
              <option v-for="anio in datosAniosFiltros" :value="anio" :key="anio">
                {{ anio }}
              </option>
            </select>
          </div>
        </div>
      </div>
    </div>
    <!-- <div class="flex justify-between align-center">
            <div v-for="mes in opcionesFiltroSegundoSemestre" :key="mes.mes">
                <input type="checkbox" class="inline-block rounded-[3px]" :id="mes.mes" v-model="mes.estaActivo">
                <label :for="mes.mes" class="inline-block">{{ mes.mes }}</label>
            </div>
        </div> -->

    <div class="mt-4 mb-2">
      <h1 class="font-bold text-2xl text-center">Informe de ventas {{ fechaFiltro }}</h1>
    </div>
    <hr>
    <section class="grid grid-cols-12 gap-2 p-2">
      <!--Grafico 1-->
      <div class="p-4  col-span-12 md:col-span-6 border mt-[1%] mb-16">
        <h3 class="font-bold text-center">Ventas Totales por mes</h3>
        <apexchart width="100%" type="bar" :options="chartOptions" :series="series"></apexchart>
      </div>
      <!--Grafico 2-->
      <div class="p-4  col-span-12 md:col-span-6 border mt-[1%] mb-16">
        <h3 class="font-bold text-center">Productos mas vendidos</h3>
        <apexchart id="chart-products" width="100%" type="bar" :options="chartOptions2" :series="series2"></apexchart>
      </div>
    </section>
    <div class="text-center">
      <h3 class="text-lg"><b>Resumen de ventas totales</b></h3>
    </div>
    <div class="w-full flex px-4">
      <div class="w-full md:w-3/4 mx-2">
        <table class="w-full m-auto rounded-lg shadow-lg mb-[2%]">
          <thead>
            <tr class="border-b-2 border-black-400 h-[40px] bg-slate-100">
              <th class=" font-bold">Mes</th>
              <th class=" font-bold">Numero de ventas</th>
              <th class=" font-bold">Monto Total Mes</th>
            </tr>
          </thead>
          <tbody class="bg-white">
            <tr class="border-b" v-for="total_ventas in datos_filtrados" :key="total_ventas.nombre_mes">
              <td class="p-2 text-center">{{ total_ventas.nombre_mes }}</td>
              <td class="p-2 text-center">{{ total_ventas.num_ventas }}</td>
              <td class="text-center">${{ formatearValorTotal(total_ventas.total_venta) }}</td>
            </tr>
          </tbody>
          <tfoot>
            <tr>
              <td class="text-center font-bold p-2">Total</td>
              <td class="text-center font-bold p-2">{{ datos_filtrados ? datos_filtrados.reduce((acumulador,
                valorActual) => acumulador + valorActual.num_ventas, 0) : 0 }}</td>
              <td class="text-center font-bold p-2">${{ datos_filtrados ? Number(datos_filtrados.reduce((acumulador,
                valorActual) => parseFloat(acumulador) + parseFloat(valorActual.total_venta), 0)).toFixed(2) : 0 }}
              </td>
            </tr>
          </tfoot>
        </table>
      </div>
      <div class="w-full md:w-1/4 text-center ">
        <div class="card shadow mb-2 py-2 bg-white">
          <p><b>Producto más vendido:</b><br>
            {{ datos_productos.length>0 ? datos_productos[0].nombre : '' }}<br>
            <span class="m-2"><b>Unidades vendidas:</b> {{ datos_productos.length>0 ? datos_productos[0].cantidad_producto : ''
              }}</span>
            <span><b>Ingresos:</b> ${{ datos_productos.length>0 ? datos_productos[0].ingresos_producto : '' }}</span>
          </p>
        </div>

        <div class="card shadow mb-2 py-2 bg-white">
          <p><b>Promedio de ventas mensual</b>
            ${{ Number(calcularPromedioVentas()).toFixed(2) }}
          </p>
        </div>
        <div class="card shadow mb-2 py-2 bg-white">
          <p><b>Numero de ventas promedio mensual</b>
            {{ Number(calcularNumPromVentas()).toFixed(2) }}
          </p>
        </div>
      </div>
    </div>
  </main>
</template>
<script>
import axios from 'axios'
import NavBar from '@/components/NavBar.vue'
export default {
  components: {
    NavBar
  },
  mounted() {
    this.obtenerFechasFiltro()
    this.obtenerDatosFiltrados()
  },
  data() {
    return {
      opcionesFiltroMeses: [
        { mes: 'ene', estaActivo: true, numMes: 2 },
        { mes: 'feb', estaActivo: true, numMes: 1 },
        { mes: 'mar', estaActivo: true, numMes: 3 },
        { mes: 'abr', estaActivo: true, numMes: 4 },
        { mes: 'may', estaActivo: true, numMes: 5 },
        { mes: 'jun', estaActivo: true, numMes: 6 },
        { mes: 'jul', estaActivo: true, numMes: 7 },
        { mes: 'ago', estaActivo: true, numMes: 8 },
        { mes: 'sep', estaActivo: true, numMes: 9 },
        { mes: 'oct', estaActivo: true, numMes: 10 },
        { mes: 'nov', estaActivo: true, numMes: 11 },
        { mes: 'dic', estaActivo: true, numMes: 12 }
      ],
      numero_ventas: 0,
      monto_total: 0,
      datos_filtrados: [],
      datos_productos: [],
      fechaFiltro: new Date().getFullYear(),
      datosAniosFiltros: [],
      chartOptions: {
        chart: {
          id: 'vuechart-example',
          height: '250px'
        },
        yaxis: {
          title: {
            text: 'Total Ventas ($)'
          }
        },
        xaxis: {
          title: {
            text: 'Meses'
          },
          categories: []
        },
        colors: '#13C296',
        plotOptions: {
          bar: {
            columnWidth: '30%',
            horizontal: false
          }
        },
        dataLabels: {
          enabled: true,
          formatter: function (val) {
            return '$' + val
          },
          style: {
            fontSize: '12px',
            fontWeight: 1000,
            colors: ['#333', '#999']
          }
        }
      },
      series: [
        {
          name: 'Ventas totales ($)',
          data: []
        }
      ],
      series2: [
        {
          name: 'Ingresos($)',
          data: [

          ],
          isPrice: true
        },
        {
          name: "Unidades vendidas",
          data: [0],
          isPrice: false
        }
      ],
      chartOptions2: {
        chart: {
          type: "bar",
          height: '250px'
        },
        plotOptions: {
          bar: {
            columnWidth: '30%',
            horizontal: true,
            dataLabels: {
              position: "top"
            },
          }
        },
        dataLabels: {
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
            text: 'Cantidades'
          },
          categories: [

          ]
        },
        yaxis: {
          title: {
            text: 'Productos'
          }
        },
      }
    }
  },
  methods: {
    obtenerFechasFiltro() {
      axios
        .get('/api/fechas_filtro')
        .then((response) => {
          this.datosAniosFiltros = response.data.lista_fechas_filtro
          //this.obtenerDatosFiltrados();
        })
        .catch((response) => {
          alert(response.response.data)
        })
    },
    construirEstructuraFiltro() {
      let filtro_meses = []
      this.opcionesFiltroMeses.forEach((filtroMes) => {
        if (filtroMes.estaActivo === true) {
          filtro_meses.push(filtroMes.numMes)
        }
      })
      return filtro_meses
    },
    obtenerDatosFiltrados() {
      const filtro_meses = this.construirEstructuraFiltro()
      //const parametros = {"filtro_meses":filtro_meses,"anio_filtro":this.fechaFiltro};
      axios
        .get('/api/filtro_ventas_totales', {
          params: {
            anio_filtro: this.fechaFiltro,
            filtro_meses: filtro_meses
          }
        })
        .then((response) => {
          this.clearData();
          //Grafico 1
          this.datos_filtrados = response.data.datos_filtrados
          this.chartOptions.xaxis.categories.splice(0, this.chartOptions.xaxis.categories.length)
          this.series[0].data.splice(0, this.series[0].data.length)
          this.datos_filtrados.forEach((totalMes) => {
            this.chartOptions.xaxis.categories.push(totalMes.nombre_mes)
            this.series[0].data.push(totalMes.total_venta)
          })
          console.log(this.datos_filtrados);
          //Grafico 2
          this.datos_productos = response.data.datos_productos
          this.chartOptions2.xaxis.categories.splice(0, this.chartOptions2.xaxis.categories.length)
          this.series2[0].data.splice(0, this.series2[0].data.length)
          this.series2[1].data.splice(0, this.series2[1].data.length)

          this.datos_productos.forEach((producto) => {
            console.log(producto.nombre)
            this.chartOptions2.xaxis.categories.push(producto.nombre)

            this.series2[0].data.push(producto.ingresos_producto)
            this.series2[1].data.push(producto.cantidad_producto)
          })

          console.log(this.chartOptions2.xaxis.categories)

          this.numero_ventas = response.data.numero_ventas[0].cantidad_ventas
          this.cantidad_ventas = response.data.numero_ventas[0].total_ventas
          console.log('Numero de ventas: ' + this.numero_ventas)
          console.log('Cantidad de ventas: ' + this.cantidad_ventas)
          //console.log(this.chartOptions)
        })
        .catch((response) => {
          console.log(response)
          this.clearData();
        })
    },
    formatearValorTotal(totalVenta) {
      return Number(totalVenta).toFixed(2)//.toString().replace('.', ',')
    },
    calcularTotalVentas() {
      return this.datos_filtrados ? this.datos_filtrados.reduce((acumulador,
        valorActual) => parseFloat(acumulador) + parseFloat(valorActual.total_venta), 0) : 0
    },
    calcularPromedioVentas() {
      return this.datos_filtrados ? this.calcularTotalVentas() / this.datos_filtrados.length : 0;
    },
    calcularNumPromVentas() {
      return this.datos_filtrados ? this.datos_filtrados.reduce((acumulador,
        valorActual) => parseFloat(acumulador) + parseFloat(valorActual.num_ventas), 0) / this.datos_filtrados.length : 0;
    },
    selectAllMonth() {
      this.opcionesFiltroMeses.forEach((value) => {
        value.estaActivo = true;
      })
    },
    clearData() {
      // Limpiar los datos existentes del gráfico
      this.series[0].data.splice(0, this.series[0].data.length);
      this.chartOptions.xaxis.categories.splice(0, this.chartOptions.xaxis.categories.length);

      this.series2[0].data.splice(0, this.series2[0].data.length);
      this.chartOptions2.xaxis.categories.splice(0, this.chartOptions2.xaxis.categories.length);

      this.datos_filtrados = null;
      this.datos_productos = null;
    }
  }
}
</script>
