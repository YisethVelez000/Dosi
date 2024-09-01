<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'
import { useRouter, useRoute } from 'vue-router'
import { Compra, Proveedor, InsumoCompra } from '../types'
import jsPDF from 'jspdf'
import * as XLSX from 'xlsx'
import autoTable from 'jspdf-autotable'

const route = useRoute()
const router = useRouter()

const compraId = Number(route.params.id)
const compra = ref<Compra | null>(null)
const proveedor = ref<Proveedor | null>(null)
const insumosCompra = ref<InsumoCompra[]>([])

const obtenerDetalleCompra = async () => {
  try {
    const response = await axios.get(`https://apinodedvp.onrender.com/compras/${compraId}`)
    compra.value = response.data
    proveedor.value = response.data.proveedor
    insumosCompra.value = response.data.Compras_Has_Insumos
  } catch (error) {
    console.error('Error al obtener el detalle de la compra:', error)
  }
}

onMounted(() => {
  obtenerDetalleCompra()
})

const regresar = () => {
  router.push({ name: 'compras' })
}

const generatePdf = () => {
  if (!compra.value || !proveedor.value || !insumosCompra.value.length) return

  const doc = new jsPDF()

  // Información de la compra
  doc.text('Detalle de Compra', 14, 16)
  doc.text(`Fecha de la compra: ${new Date(compra.value.FechaCompra).toLocaleDateString()}`, 14, 24)
  doc.text(`Recibo: ${compra.value.Recibo}`, 14, 32)
  doc.text(`Proveedor: ${proveedor.value.NombreProveedor}`, 14, 40)
  doc.text(`Anulada: ${compra.value.Anulada}`, 14, 48)
  if (compra.value.Anulada === 'Sí') {
    doc.text(`Motivo de Anulación: ${compra.value.MotivoAnulacion}`, 14, 56)
  }
  doc.text(`Precio total de la compra: ${compra.value.PrecioTotalCompra}`, 14, 64)

  // Imagen del recibo
  if (compra.value.ImagenRecibo) {
    const img = new Image()
    img.src = compra.value.ImagenRecibo
    img.onload = () => {
      doc.addImage(img, 'JPEG', 14, 72, 180, 160)

      // Incluir los insumos
      autoTable(doc, {
        startY: doc.autoTable.previous ? doc.autoTable.previous.finalY + 10 : 72 + 160 + 10,
        head: [['Nombre Insumo', 'Cantidad Comprada']],
        body: insumosCompra.value.map((insumo) => [
          insumo.insumo?.NombreInsumo || '',
          insumo.CantidadCompra.toString(),
        ]),
        theme: 'striped',
      })

      doc.save(`compra_${compraId}.pdf`)
    }
  } else {
    // Incluir los insumos si no hay imagen
    autoTable(doc, {
      startY: doc.autoTable.previous ? doc.autoTable.previous.finalY + 10 : 72,
      head: [['Nombre Insumo', 'Cantidad Comprada']],
      body: insumosCompra.value.map((insumo) => [insumo.insumo?.NombreInsumo || '', insumo.CantidadCompra.toString()]),
      theme: 'striped',
    })

    doc.save(`compra_${compraId}.pdf`)
  }
}

const generateExcel = () => {
  if (!compra.value || !proveedor.value || !insumosCompra.value.length) return

  // Información de la compra
  const wsCompra: XLSX.WorkSheet = XLSX.utils.json_to_sheet([
    {
      'Fecha de la compra': new Date(compra.value.FechaCompra).toLocaleDateString(),
      Recibo: compra.value.Recibo,
      Proveedor: proveedor.value.NombreProveedor,
      Anulada: compra.value.Anulada,
      'Motivo de Anulación': compra.value.Anulada === 'Sí' ? compra.value.MotivoAnulacion : '',
      'Precio total de la compra': compra.value.PrecioTotalCompra,
    },
  ])

  // Información de los insumos
  const wsInsumos: XLSX.WorkSheet = XLSX.utils.json_to_sheet(
    insumosCompra.value.map((insumo) => ({
      'Nombre Insumo': insumo.insumo?.NombreInsumo || '',
      'Cantidad Comprada': insumo.CantidadCompra,
    })),
  )

  // Crear un libro de trabajo
  const wb = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(wb, wsCompra, 'Detalles de Compra')
  XLSX.utils.book_append_sheet(wb, wsInsumos, 'Insumos')

  // Exportar a Excel
  XLSX.writeFile(wb, `compra_${compraId}.xlsx`)
}
</script>

<template>
  <div class="compra-detalle">
    <h1 class="titulo">Detalle de Compra</h1>
    <VaCard id="reporteCompra">
      <VaCardContent v-if="compra">
        <!-- Información general de la compra -->
        <table class="recibo-table">
          <tr>
            <th>Fecha de la compra</th>
            <td>{{ new Date(compra.FechaCompra).toLocaleDateString() }}</td>
          </tr>
          <tr>
            <th>Recibo</th>
            <td>{{ compra.Recibo }}</td>
          </tr>
          <tr>
            <th>Proveedor</th>
            <td>{{ proveedor?.NombreProveedor }}</td>
          </tr>
          <tr>
            <th>Anulada</th>
            <td>{{ compra.Anulada }}</td>
          </tr>
          <tr v-if="compra.Anulada === 'Sí'">
            <th>Motivo de Anulación</th>
            <td>{{ compra.MotivoAnulacion }}</td>
          </tr>
          <tr>
            <th>Precio total de la compra</th>
            <td>{{ compra.PrecioTotalCompra }}</td>
          </tr>
        </table>

        <!-- Imagen del recibo -->
        <h3 class="subtitulo">Imagen del recibo</h3>
        <div class="recibo-imagen">
          <img :src="compra.ImagenRecibo" alt="Imagen del recibo" class="recibo-img" />
        </div>

        <!-- Insumos de la compra -->
        <h3 class="subtitulo">Insumos de la compra</h3>
        <div class="insumos-detalle">
          <table class="insumos-table">
            <thead>
              <tr>
                <th>Nombre Insumo</th>
                <th>Cantidad Comprada</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="insumoCompra in insumosCompra" :key="insumoCompra.IdInsumoCompra">
                <td>{{ insumoCompra.insumo?.NombreInsumo }}</td>
                <td>{{ insumoCompra.CantidadCompra }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Botones de acción -->
        <div class="button-container">
          <VaButton color="primary" class="mt-4 button-space" @click="generatePdf">Generar PDF</VaButton>
          <VaButton color="primary" class="mt-4 button-space" @click="generateExcel">Generar Excel</VaButton>
          <VaButton color="primary" class="mt-4 button-space" @click="regresar">Regresar</VaButton>
        </div>
      </VaCardContent>
      <VaCardContent v-else>
        <p>Cargando...</p>
      </VaCardContent>
    </VaCard>
  </div>
</template>

<style scoped>
.compra-detalle {
  padding: 20px;
}

.titulo {
  text-align: center;
  font-weight: bold;
  font-size: 1.5em;
  margin-bottom: 20px;
}

.subtitulo {
  font-weight: bold;
  margin-top: 20px;
  margin-bottom: 10px;
}

.recibo-table,
.insumos-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

.recibo-table th,
.recibo-table td,
.insumos-table th,
.insumos-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

.recibo-table th,
.insumos-table th {
  background-color: #f2f2f2;
  font-weight: bold;
}

.recibo-imagen {
  margin-top: 10px;
  text-align: center;
}

.recibo-img {
  max-width: 200px;
  max-height: 200px;
  border: 1px solid #ddd;
  margin-top: 10px;
}

.recibo-actions {
  text-align: center;
  margin-top: 20px;
}

.button-container {
  display: flex;
  gap: 10px;
  justify-content: flex-start;
}
.button-space {
  margin-right: 10px;
}
</style>
