<template>
    <main class="relative">
        <NavBar />

        <!-- Encabezado -->
        <div>
            <div class="flex bg-white mx-auto p-5 shadow-md justify-between">
                <h1 class="font-bold text-blue-700 text-xl">Informe de Pedidos a domicilio</h1>
            </div>
            <div class="flex justify-start items-center mt-4 ml-4">
                <a href="#" @click="$router.go(-1)" class="text-sm text-black font-medium flex items-center">
                    <img src="../../assets/icons/arrow.svg" alt="Regresar" class="h-6 w-6 mr-1" /> Regresar
                </a>
            </div>
        </div>

        <div class="grid grid-cols-10 gap-4 w-[90%] m-auto">
            <div class="col-span-8 flex align-center">
                <div class="grid grid-cols-6 gap-4 w-[45%] ml-[10%] mt-[4%]">
                    <div v-for="mes in opcionesFiltroMeses" :key="mes.mes">
                        <input type="checkbox" class="inline-block rounded-[3px]" :id="mes.mes"
                            v-model="mes.estaActivo" />
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
                            @click="getDatosPedidosDomicilio">
                            Aplicar
                        </button>
                    </div>
                    <div class="p-4 mr-[10%]">
                        <select name="filtroAnio" id="filtroAnio"
                            class="border-slate-300 rounded text-slate-400 bg-slate-50" v-model="fechaFiltro">
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
        <!--Contenedor de graficos-->
        <div class="my-4"><h1 class="text-center text-3xl">Informe de Pedidos Mensuales {{ fechaFiltro }}</h1></div><hr>
        <section class="grid grid-cols-12 w-full mt-4">
            <!--Grafico1-->
            <div class="boder md:col-span-6 col-span-12">
                <div class="p-4 w-[80%] col-span-6 m-auto border mt-[1%] mb-16">
                    <h3 class="text-center">Numero de Pedidos por Mes</h3>
                    <apexchart width="100%" height="150%" type="bar" :options="chartOptions" :series="series">
                    </apexchart>
                </div>
            </div>
            <!--Grafico2-->
            <div class="boder md:col-span-6 col-span-12">
                <div class="p-4 w-[80%] col-span-6 m-auto border mt-[1%] mb-16">
                    <h3 class="text-center">Total Ingresos Mensuales</h3>
                    <apexchart width="100%" height="150%" type="bar" :options="chartOptions2" :series="series2">
                    </apexchart>
                </div>
            </div>
        </section>

        <div class="text-center">
            <h3><b>Resumen de Pedidos a Domicilio</b></h3>
        </div>
        <div class="w-full flex px-4">
            <div class="w-full md:w-3/4 mx-2">
                <table class="w-full m-auto rounded-lg shadow-lg mb-[2%]">
                    <thead>
                        <tr class="border-b-2 border-black-400 h-[40px] bg-slate-100">
                            <th class=" font-bold">Mes</th>
                            <th class=" font-bold">Pedidos</th>
                            <th class="font-bold">Rutas</th>
                            <th class="font-bold">Ingresos</th>
                        </tr>
                    </thead>
                    <tbody class="bg-white">
                        <tr class="border-b" v-for="row in datos_pedidos" :key="datos_pedidos.mes">
                            <td class="p-[1.5%] text-center">{{ meses[row.mes - 1] }}</td>
                            <td class="text-center">{{ row.total_ventas_domicilio }}</td>
                            <td class="text-center">{{ row.total_hojas_ruta }}</td>
                            <td class="text-center">${{ row.ingresos }}</td>
                        </tr>
                    </tbody>
                    <tfoot>
                        <tr>
                            <td class="text-center font-bold p-2">Total</td>
                            <td class="text-center font-bold p-2">{{ datos_pedidos ? datos_pedidos.reduce((acumulador,
                                valorActual) => acumulador + valorActual.total_ventas_domicilio, 0) : 0 }}</td>
                            <td class="text-center font-bold p-2">{{ datos_pedidos ? datos_pedidos.reduce((acumulador,
                                valorActual) => acumulador + valorActual.total_hojas_ruta, 0) : 0 }}</td>
                            <td class="text-center font-bold p-2">${{ datos_pedidos ? Number(datos_pedidos.reduce((acumulador,
                                valorActual) => parseFloat(acumulador) + parseFloat(valorActual.ingresos), 0)).toFixed(2) : 0 }}
                            </td>
                        </tr>
                    </tfoot>
                </table>
            </div>
            <div class="w-full md:w-1/4 text-center ">
                <div class="card shadow mb-2 py-2 bg-white">
                    <p><b>Promedio de Pedidos Mensuales:</b>
                        {{ prom_pedidos_mensuales }}
                    </p>
                </div>

                <div class="card shadow mb-2 py-2 bg-white">
                    <p><b>Ingresos Promedio Mensuales:</b>
                        ${{ Number(prom_ingresos_mensuales).toFixed(2) }}
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
        //this.obtenerDatosFiltrados()
        this.getDatosPedidosDomicilio()
    },
    data() {
        return {
            meses: [
                'Enero',
                'Febrero',
                'Marzo',
                'Abril',
                'Mayo',
                'Junio',
                'Julio',
                'Agosto',
                'Septiembre',
                'Octubre',
                'Noviembre',
                'Diciembre'
            ],
            opcionesFiltroMeses: [
                { mes: 'ene', estaActivo: true, numMes: 1 },
                { mes: 'feb', estaActivo: true, numMes: 2 },
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
            datos_filtrados: null,
            datos_pedidos: null,
            prom_pedidos_mensuales: 0,
            prom_hojas_ruta: 0,
            prom_ingresos_mensuales: 0,
            fechaFiltro: new Date().getFullYear(),
            datosAniosFiltros: [],
            chartOptions: {
                chart: {
                    id: 'vuechart-example',
                    stackType: '100%'
                },
                yaxis: {
                    title: {
                        text: 'Total Pedidos'
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
                        return Number(val).toFixed(0)
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
                    name: 'Pedidos totales',
                    data: []
                }
            ],

            chartOptions2: {
                chart: {
                    id: 'vuechart-example',
                    stackType: '100%'
                },
                yaxis: {
                    title: {
                        text: 'Ingresos ($)'
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
                        return `$ ${Number(val).toFixed(2)}`
                    },
                    style: {
                        fontSize: '12px',
                        fontWeight: 1000,
                        colors: ['#333', '#999']
                    }
                }
            },
            series2: [
                {
                    name: 'Ingresos por pedidos ($)',
                    data: []
                }
            ]
        }
    },
    methods: {
        getDatosPedidosDomicilio() {
            const filtro_meses = this.construirEstructuraFiltro()
            let params = {
                anio_filtro: this.fechaFiltro,
                filtro_meses: filtro_meses
            }

            axios.get('/api/informe_pedidos_domicilio', {
                params: {
                    anio_filtro: this.fechaFiltro,
                    filtro_meses: filtro_meses
                }
            }).then(
                (response) => {
                    console.log(response.data)
                    this.clearData();
                    this.datos_pedidos = response.data.datos_pedidos
                    this.series[0].data.splice(0, this.series[0].data.length)
                    this.series2[0].data.splice(0,this.series2[0].data.length)
                    this.datos_pedidos.forEach((row) => {
                        //Grafico1
                        this.chartOptions.xaxis.categories.push(this.opcionesFiltroMeses[row.mes - 1].mes)
                        this.series[0].data.push(row.total_ventas_domicilio)
                        //Grafico2
                        this.chartOptions2.xaxis.categories.push(this.opcionesFiltroMeses[row.mes - 1].mes)
                        this.series2[0].data.push(row.ingresos)
                    })

                    this.prom_hojas_ruta = response.data.prom_hojas_ruta;
                    this.prom_pedidos_mensuales = response.data.prom_pedidos_mensuales;
                    this.prom_ingresos_mensuales = response.data.prom_ingresos_mensuales;

                }
            ).catch((error) => {
                this.clearData();
                console.log(error)
            })
        },
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
        formatearValorTotal(totalVenta) {
            return Number(totalVenta).toFixed(2)//.toString().replace('.', ',')
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
            
            this.prom_hojas_ruta = 0;
            this.prom_ingresos_mensuales = 0;
            this.prom_pedidos_mensuales = 0;
            this.datos_pedidos = []
        }
    }
}
</script>