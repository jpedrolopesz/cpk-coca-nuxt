<template>
    <div class="flex">
        <div
            class="p-6 bg-white border border-gray-300 rounded-2xl shadow-lg w-full"
        >
            <h2 class="text-xl font-bold text-gray-800 mb-4 text-center">
                Monitoramento de CPK e Tendências
            </h2>

            <div class="flex justify-center items-center py-4">
                <canvas ref="chartRef" class="w-full"></canvas>
            </div>

            <div class="mt-6">
                <h2
                    class="text-lg font-semibold text-gray-700 mb-3 border-b pb-2"
                >
                    Estatísticas
                </h2>
                <ul class="space-y-3 text-gray-600 text-sm">
                    <li class="flex justify-between">
                        <span class="font-medium">Média:</span>
                        <span>{{ media.toFixed(2) }}</span>
                    </li>
                    <li class="flex justify-between">
                        <span class="font-medium">Desvio Padrão (σ):</span>
                        <span>{{ desvioPadrao.toFixed(2) }}</span>
                    </li>
                    <li class="flex justify-between">
                        <span class="font-medium">CPK:</span>
                        <span>{{ cpk.toFixed(2) }}</span>
                    </li>
                    <li class="flex justify-between">
                        <span class="font-medium">CP:</span>
                        <span>{{ cp.toFixed(2) }}</span>
                    </li>
                    <li class="flex justify-between">
                        <span class="font-medium">Amplitude (Máx - Mín):</span>
                        <span>{{ amplitude.toFixed(2) }}</span>
                    </li>
                </ul>
            </div>
        </div>

        <div class="flex justify-center mb-6">
            <RangeCalendar v-model="selectedRange" class="rounded-md border" />
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from "vue";
import { RangeCalendar } from "~/components/ui/range-calendar"; // Corrige importação para Nuxt
import { getLocalTimeZone, today } from "@internationalized/date";
import type { DateRange } from "radix-vue";
import {
    Chart,
    LineController,
    CategoryScale,
    LinearScale,
    PointElement,
    LineElement,
    Title,
    Tooltip,
    Legend,
} from "chart.js";

// **Registrar componentes do Chart.js**
Chart.register(
    LineController,
    CategoryScale,
    LinearScale,
    PointElement,
    LineElement,
    Title,
    Tooltip,
    Legend,
);

// **Receber os dados via props**
const props = defineProps<{
    cpkData: Array<{
        data: string;
        valor: number;
        valor_minimo: number;
        valor_maximo: number;
        valor_alvo: number;
    }>;
}>();

// **Referência para o gráfico**
const chartRef = ref<HTMLCanvasElement | null>(null);
let chartInstance: Chart | null = null;

// **Intervalo de Datas Selecionado**
const start = today(getLocalTimeZone());
const end = start.add({ days: 7 });
const selectedRange = ref<DateRange>({ start, end });

/**
 * Filtra os dados conforme o intervalo de datas selecionado.
 */

const getFilteredData = computed(() => {
    return (
        props.cpkData?.filter((data) => {
            return (
                props.cpkData?.filter((data) => {
                    const dataDate = new Date(data.data);
                    const startDate = new Date(
                        selectedRange.value.start.year,
                        selectedRange.value.start.month - 1,
                        selectedRange.value.start.day,
                    );
                    const endDate = new Date(
                        selectedRange.value.end.year,
                        selectedRange.value.end.month - 1,
                        selectedRange.value.end.day,
                    );

                    return dataDate >= startDate && dataDate <= endDate;
                }) || []
            );
        }) || []
    );
});

// **Cálculo das Estatísticas**
const media = computed(() =>
    getFilteredData.value.length
        ? getFilteredData.value.reduce((sum, d) => sum + d.valor, 0) /
          getFilteredData.value.length
        : 0,
);
const desvioPadrao = computed(() => {
    if (!getFilteredData.value.length) return 0;
    const mean = media.value;
    return Math.sqrt(
        getFilteredData.value.reduce(
            (sum, d) => sum + Math.pow(d.valor - mean, 2),
            0,
        ) / getFilteredData.value.length,
    );
});
const minValor = computed(() =>
    getFilteredData.value.length
        ? Math.min(...getFilteredData.value.map((d) => d.valor_minimo))
        : 0,
);
const maxValor = computed(() =>
    getFilteredData.value.length
        ? Math.max(...getFilteredData.value.map((d) => d.valor_maximo))
        : 1,
);
const amplitude = computed(() => maxValor.value - minValor.value);
const cp = computed(() =>
    desvioPadrao.value === 0
        ? 0
        : (maxValor.value - minValor.value) / (6 * desvioPadrao.value),
);
const cpk = computed(() =>
    desvioPadrao.value === 0
        ? 0
        : Math.min(
              (media.value - minValor.value) / (3 * desvioPadrao.value),
              (maxValor.value - media.value) / (3 * desvioPadrao.value),
          ),
);

/**
 * Atualiza o gráfico ao mudar o intervalo de datas.
 */
const updateChart = () => {
    if (!chartRef.value) return;
    if (!getFilteredData.value.length) return;

    const labels = getFilteredData.value.map((d) =>
        new Date(d.data).toLocaleDateString("pt-BR"),
    );
    const values = getFilteredData.value.map((d) => d.valor);
    const cpkAlvo = getFilteredData.value.map((d) => d.valor_alvo);
    const valoresMinimos = getFilteredData.value.map((d) => d.valor_minimo);
    const valoresMaximos = getFilteredData.value.map((d) => d.valor_maximo);

    if (chartInstance) chartInstance.destroy();
    chartInstance = new Chart(chartRef.value, {
        type: "line",
        data: {
            labels,
            datasets: [
                {
                    label: "Valores Monitorados",
                    data: values,
                    borderColor: "blue",
                    borderWidth: 2,
                    fill: false,
                },
                {
                    label: "CPK Alvo",
                    data: cpkAlvo,
                    borderColor: "orange",
                    borderDash: [5, 5],
                    pointRadius: 0,
                    fill: false,
                },
                {
                    label: "Valor Mínimo",
                    data: valoresMinimos,
                    borderColor: "red",
                    borderDash: [5, 5],
                    pointRadius: 0,
                    fill: false,
                },
                {
                    label: "Valor Máximo",
                    data: valoresMaximos,
                    borderColor: "green",
                    borderDash: [5, 5],
                    pointRadius: 0,
                    fill: false,
                },
            ],
        },
        options: {
            responsive: true,
            scales: { y: { min: minValor.value, max: maxValor.value } },
        },
    });
};

// Atualiza o gráfico ao mudar o intervalo de datas
watch(selectedRange, updateChart, { deep: true });
onMounted(updateChart);
</script>
