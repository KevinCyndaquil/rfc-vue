<script>
import axios from 'axios';

export default {
    data() {
        return {
            productData: null, // Para un solo producto
            productId: '', // ID del producto para búsqueda
            products: [], // Lista de todos los productos
            filters: {
                global: { value: null } // Filtro global
            },
            loading: false, // Estado de carga para la búsqueda por ID
            loadingAll: true // Estado de carga para todos los productos
        };
    },
    methods: {
        // Buscar un producto por ID
        searchProduct() {
            this.loading = true;
            axios
                .get(`https://api.restful-api.dev/objects/${this.productId}`)
                .then((response) => {
                    if (response.data) {
                        this.productData = {
                            id: response.data.id,
                            name: response.data.name || 'N/A',
                            color: response.data.data?.color || 'N/A',
                            capacity: response.data.data?.capacity || response.data.data?.['capacity GB'] || 'N/A'
                        };
                    } else {
                        this.productData = null;
                    }
                })
                .catch((error) => {
                    console.error('Error al buscar el producto:', error);
                    this.productData = null;
                })
                .finally(() => {
                    this.loading = false;
                });
        },

        // Cargar todos los productos
        loadProducts() {
            this.loadingAll = true;
            axios
                .get('https://api.restful-api.dev/objects')
                .then((response) => {
                    if (Array.isArray(response.data)) {
                        this.products = response.data.map((item) => ({
                            id: item.id,
                            name: item.name || 'N/A',
                            color: item.data?.color || 'N/A',
                            capacity: item.data?.capacity || item.data?.['capacity GB'] || 'N/A'
                        }));
                    } else {
                        this.products = [];
                    }
                })
                .catch((error) => {
                    console.error('Error al cargar los productos:', error);
                })
                .finally(() => {
                    this.loadingAll = false;
                });
        },

        // Limpiar el filtro global
        clearFilter() {
            this.filters = { global: { value: null } };
        }
    },
    mounted() {
        this.loadProducts(); // Cargar todos los productos al montar el componente
    }
};
</script>

<template>
    <div class="flex flex-col gap-6 p-4">
        <!-- Búsqueda por ID -->
        <div class="bg-white p-6 rounded-lg shadow-md max-w-lg mx-auto">
            <div class="font-semibold text-xl mb-4">Buscar Producto</div>
            <div class="flex flex-col gap-2 mb-4">
                <label for="productId">ID del Producto:</label>
                <InputText id="productId" v-model="productId" type="text" />
                <Button label="Buscar" @click="searchProduct" />
            </div>

            <!-- Detalles del Producto -->
            <div v-if="productData" class="mt-6">
                <div class="font-semibold text-xl mb-4">Detalles del Producto</div>
                <div class="flex flex-col gap-4">
                    <div class="flex items-center">
                        <label for="name" class="mr-4 w-32">Nombre:</label>
                        <InputText id="name" v-model="productData.name" type="text" class="flex-grow" disabled />
                    </div>
                    <div class="flex items-center">
                        <label for="color" class="mr-4 w-32">Color:</label>
                        <InputText id="color" v-model="productData.color" type="text" class="flex-grow" disabled />
                    </div>
                    <div class="flex items-center">
                        <label for="capacity" class="mr-4 w-32">Capacidad:</label>
                        <InputText id="capacity" v-model="productData.capacity" type="text" class="flex-grow" disabled />
                    </div>
                </div>
            </div>

            <div v-if="loading" class="mt-4 text-center">Cargando...</div>
        </div>

        <!-- Tabla de productos -->
        <div class="card">
            <div class="font-semibold text-xl mb-4">Productos Disponibles</div>
            <DataTable :value="products" :paginator="true" :rows="10" :loading="loadingAll" :filters="filters" filterDisplay="menu" :globalFilterFields="['name', 'color', 'capacity']" showGridlines dataKey="id">
                <!-- Filtro Global -->
                <template #header>
                    <div class="flex justify-between">
                        <InputText v-model="filters.global.value" placeholder="Buscar productos..." />
                    </div>
                </template>

                <!-- Columnas de la tabla -->
                <Column field="id" header="ID" style="min-width: 6rem" sortable></Column>
                <Column field="name" header="Nombre" style="min-width: 12rem" sortable></Column>
                <Column field="color" header="Color" style="min-width: 12rem" sortable></Column>
                <Column field="capacity" header="Capacidad" style="min-width: 12rem" sortable></Column>

                <!-- Mensajes personalizados -->
                <template #empty>No se encontraron productos.</template>
                <template #loading>Cargando productos, por favor espere...</template>
            </DataTable>
        </div>
    </div>
</template>

<style scoped>
/* Estilos generales */
.card {
    background-color: white;
    border-radius: 15px;
    box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
    padding: 1rem;
}
</style>
