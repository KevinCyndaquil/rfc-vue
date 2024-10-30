<script setup>
import { ref } from 'vue';

const nombre = ref('');
const apellidoPaterno = ref('');
const apellidoMaterno = ref('');
const fechaNacimiento = ref('');
const rfc = ref('');
const loading = ref(false);
const colorClase = ref('color-inicial'); // Estado inicial

function generarRFC() {
    loading.value = true;

    // Cambiar color antes de procesar el RFC para que sea visible inmediatamente
    colorClase.value = colorClase.value === 'color-inicial' ? 'color-alternativo' : 'color-inicial';

    setTimeout(() => {
        if (nombre.value && apellidoPaterno.value && apellidoMaterno.value && fechaNacimiento.value) {
            const primerasLetrasPaterno = apellidoPaterno.value.slice(0, 2).toUpperCase();
            const primeraLetraMaterno = apellidoMaterno.value.charAt(0).toUpperCase();
            const primeraLetraNombre = nombre.value.charAt(0).toUpperCase();
            const fecha = new Date(fechaNacimiento.value);
            const anio = fecha.getFullYear().toString().slice(-2);
            const mes = (fecha.getMonth() + 1).toString().padStart(2, '0');
            const dia = fecha.getDate().toString().padStart(2, '0');

            const caracteresRandom = () => {
                const caracteres = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
                return Array.from({ length: 3 }, () => caracteres.charAt(Math.floor(Math.random() * caracteres.length))).join('');
            };

            rfc.value = `${primerasLetrasPaterno}${primeraLetraMaterno}${primeraLetraNombre}${anio}${mes}${dia}${caracteresRandom()}`;
        } else {
            rfc.value = 'Por favor, completa todos los campos.';
        }
        loading.value = false;
    }, 1000);
}
</script>

<template>
    <Fluid>
        <div class="flex flex-col md:flex-row gap-8">
            <div class="md:w-1/2">
                <!-- Aplicamos :class="colorClase" para cambiar la clase y el color -->
                <div class="card flex flex-col gap-4" :class="colorClase">
                    <div class="font-semibold text-xl">Generar RFC</div>

                    <div class="flex flex-col gap-2">
                        <label for="nombre">Nombre</label>
                        <InputText id="nombre" v-model="nombre" type="text" />
                    </div>

                    <div class="flex flex-col gap-2">
                        <label for="apellidoPaterno">Apellido Paterno</label>
                        <InputText id="apellidoPaterno" v-model="apellidoPaterno" type="text" />
                    </div>

                    <div class="flex flex-col gap-2">
                        <label for="apellidoMaterno">Apellido Materno</label>
                        <InputText id="apellidoMaterno" v-model="apellidoMaterno" type="text" />
                    </div>

                    <div class="flex flex-col gap-2">
                        <label for="fechaNacimiento">Fecha de Nacimiento</label>
                        <InputText id="fechaNacimiento" v-model="fechaNacimiento" type="date" />
                    </div>

                    <div class="flex flex-wrap gap-2 mt-4">
                        <Button label="Generar RFC" icon="pi pi-check" :loading="loading" @click="generarRFC" />
                    </div>

                    <div class="font-semibold text-xl">RFC Generado</div>
                    <div class="border p-4 rounded">{{ rfc }}</div>
                </div>
            </div>
        </div>
    </Fluid>
</template>

<style scoped>
.color-inicial {
    background-color: #ffffff; /* Blanco */
    color: #333333; /* Texto oscuro */
}

.color-alternativo {
    background-color: #4caf50; /* Verde */
    color: #ffffff; /* Texto blanco */
}
</style>
