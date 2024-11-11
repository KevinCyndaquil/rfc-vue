<script setup>
import Button from 'primevue/button';
import Column from 'primevue/column';
import DataTable from 'primevue/datatable';
import Dialog from 'primevue/dialog';
import InputNumber from 'primevue/inputnumber';
import InputText from 'primevue/inputtext';
import Toast from 'primevue/toast';
import { ref } from 'vue';

const productos = ref([]);
const nuevoProducto = ref({
    nombre: '',
    precioUnitario: 0,
    cantidad: 1
});
const totalSinIva = ref(0);
const iva = ref(0);
const totalConIva = ref(0);

const toast = ref(null);
const visibleUpdate = ref(false);
const visibleDelete = ref(false);
const productoSeleccionado = ref(null);
const nomp = ref('');
const precioUnitario = ref(0);
const cantidad = ref(1);

function agregarProducto() {
    if (nuevoProducto.value.nombre && nuevoProducto.value.precioUnitario > 0) {
        const producto = {
            consecutivo: productos.value.length + 1,
            nombre: nuevoProducto.value.nombre,
            precioUnitario: nuevoProducto.value.precioUnitario,
            cantidad: nuevoProducto.value.cantidad,
            precioTotal: nuevoProducto.value.precioUnitario * nuevoProducto.value.cantidad
        };
        productos.value.push(producto);
        calcularTotales();
        toast.value.add({ severity: 'success', summary: 'Producto agregado', detail: `El producto "${producto.nombre}" se ha agregado correctamente.`, life: 3000 });
        nuevoProducto.value.nombre = '';
        nuevoProducto.value.precioUnitario = 0;
        nuevoProducto.value.cantidad = 1;
    } else {
        toast.value.add({ severity: 'error', summary: 'Error', detail: 'Por favor, ingrese un nombre y precio válido.', life: 3000 });
    }
}

function calcularTotales() {
    totalSinIva.value = productos.value.reduce((sum, producto) => sum + producto.precioTotal, 0);
    iva.value = totalSinIva.value * 0.16;
    totalConIva.value = totalSinIva.value + iva.value;
}

function editarProducto(producto) {
    productoSeleccionado.value = producto;
    nomp.value = producto.nombre;
    precioUnitario.value = producto.precioUnitario;
    cantidad.value = producto.cantidad;
    visibleUpdate.value = true;
}

function updateProducto() {
    if (productoSeleccionado.value) {
        productoSeleccionado.value.nombre = nomp.value;
        productoSeleccionado.value.precioUnitario = precioUnitario.value;
        productoSeleccionado.value.cantidad = cantidad.value;
        productoSeleccionado.value.precioTotal = precioUnitario.value * cantidad.value;
        calcularTotales();
        visibleUpdate.value = false;
        toast.value.add({ severity: 'success', summary: 'Producto actualizado', detail: `El producto "${productoSeleccionado.value.nombre}" se ha actualizado correctamente.`, life: 3000 });
    }
}

// Eliminar un producto
function eliminarProducto(producto) {
    productoSeleccionado.value = producto;
    visibleDelete.value = true;
}

// Confirmar eliminación del producto
function deleteProducto() {
    if (productoSeleccionado.value) {
        productos.value = productos.value.filter((producto) => producto !== productoSeleccionado.value);
        calcularTotales();
        visibleDelete.value = false;
        toast.value.add({ severity: 'success', summary: 'Producto eliminado', detail: `El producto "${productoSeleccionado.value.nombre}" ha sido eliminado.`, life: 3000 });
    }
}
</script>

<template>
    <div>
        <div class="mb-4 flex flex-col md:flex-row gap-4">
            <div class="flex flex-col w-full md:w-1/4">
                <label for="nombre">Nombre</label>
                <InputText v-model="nuevoProducto.nombre" id="nombre" type="text" placeholder="Nombre del producto" />
            </div>
            <div class="flex flex-col w-full md:w-1/4">
                <label for="precioUnitario">Precio Unitario</label>
                <InputNumber v-model="nuevoProducto.precioUnitario" id="precioUnitario" type="number"
                    placeholder="Precio unitario" />
            </div>
            <div class="flex flex-col w-full md:w-1/4">
                <label for="cantidad">Cantidad</label>
                <InputNumber v-model="nuevoProducto.cantidad" id="cantidad" type="number" min="1"
                    placeholder="Cantidad" />
            </div>
            <div class="flex items-end justify-end w-full md:w-1/4">
                <Button label="Agregar Producto" icon="pi pi-plus" @click="agregarProducto" />
            </div>
        </div>

        <!-- DataTable de productos -->
        <div class="mb-4">
            <DataTable :value="productos" responsiveLayout="scroll" class="p-datatable-striped">
                <Column field="consecutivo" header="Consecutivo" />
                <Column field="nombre" header="Nombre" />
                <Column field="precioUnitario" header="Precio Unitario" />
                <Column field="cantidad" header="Cantidad" />
                <Column field="precioTotal" header="Precio Total" />
                <Column style="width: 140px" header="Acciones">
                    <template #body="slotProps">
                        <Button icon="pi pi-pencil" type="button" class="p-button-success p-mr-2 p-mb-1"
                            @click="editarProducto(slotProps.data)" />
                        <Dialog v-model:visible="visibleUpdate" modal header="Actualizar datos de un producto"
                            :style="{ width: '45vw' }">
                            <div class="flex gap-2">
                                <div class="flex-auto">
                                    <label for="nomp"><strong>Nombre del producto: </strong></label>
                                    <InputText class="ml-2" id="nomp" v-model="nomp" />
                                </div>
                                <div class="flex-auto">
                                    <label for="precioUnitario"><strong>Precio Unitario: </strong></label>
                                    <InputNumber class="ml-2" id="precioUnitario" v-model="precioUnitario" />
                                </div>
                                <div class="flex-auto">
                                    <label for="cantidad"><strong>Cantidad: </strong></label>
                                    <InputNumber class="ml-2" id="cantidad" v-model="cantidad" />
                                </div>
                            </div>
                            <template #footer>
                                <Button label="Actualizar" icon="pi pi-replay" @click="updateProducto" />
                            </template>
                        </Dialog>

                        <Button icon="pi pi-trash" type="button" class="p-button-danger p-mb-1"
                            @click="eliminarProducto(slotProps.data)" />
                        <Dialog v-model:visible="visibleDelete" modal
                            header="¿Estás seguro que quieres borrar este producto?" :style="{ width: '30vw' }">
                            <template #footer>
                                <Button label="Eliminar" severity="warning" icon="pi pi-check" @click="deleteProducto"
                                    autofocus />
                            </template>
                        </Dialog>
                    </template>
                </Column>
            </DataTable>
        </div>

        <!-- Totales -->
        <div class="mt-4 p-4 border border-t-0 rounded-md">
            <div class="flex justify-between">
                <span>Total sin IVA:</span>
                <span>{{ totalSinIva.toFixed(2) }} MXN</span>
            </div>
            <div class="flex justify-between">
                <span>IVA (16%):</span>
                <span>{{ iva.toFixed(2) }} MXN</span>
            </div>
            <div class="flex justify-between font-semibold">
                <span>Total con IVA:</span>
                <span>{{ totalConIva.toFixed(2) }} MXN</span>
            </div>
        </div>

        <!-- Toastr -->
        <Toast ref="toast" />
    </div>
</template>

<style scoped>
.table {
    width: 100%;
    border-collapse: collapse;
}

th,
td {
    padding: 8px;
    text-align: left;
    border: 1px solid #ddd;
}

th {
    background-color: #f4f4f4;
}
</style>
