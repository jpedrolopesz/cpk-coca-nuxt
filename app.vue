<template>
    <div class="flex flex-col md:flex-row gap-4">
        <!-- Gráfico + Estatísticas -->
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

        <!-- Upload + Tabela CSV + RangeCalendar -->
        <div class="flex flex-col gap-4 w-full">
            <div
                class="p-6 bg-white border border-gray-300 rounded-2xl shadow-lg"
            >
                <h2 class="text-xl font-bold text-gray-800 mb-4 text-center">
                    Upload e Processamento de CSV
                </h2>
                <input
                    type="file"
                    @change="handleFileUpload"
                    accept=".csv"
                    class="mb-4"
                />

                <!-- Exibe dados importados em tabela, para conferência -->
                <div v-if="parsedData.length" class="mt-4">
                    <h3 class="text-lg font-semibold text-gray-700 mb-3">
                        Dados Importados:
                    </h3>
                    <div
                        class="overflow-auto max-h-60 border border-gray-200 rounded-md p-2"
                    >
                        <table class="w-full text-sm text-left border-collapse">
                            <thead class="bg-gray-100">
                                <tr>
                                    <th
                                        v-for="(header, index) in headers"
                                        :key="index"
                                        class="border p-2"
                                    >
                                        {{ header }}
                                    </th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr
                                    v-for="(row, rowIndex) in parsedData"
                                    :key="rowIndex"
                                    class="border"
                                >
                                    <td
                                        v-for="(value, key) in row"
                                        :key="key"
                                        class="p-2 border"
                                    >
                                        {{ value }}
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>

            <!-- RangeCalendar para filtrar por datas -->
            <div
                class="p-6 bg-white border border-gray-300 rounded-2xl shadow-lg"
            >
                <h2 class="text-xl font-bold text-gray-800 mb-4 text-center">
                    Filtrar por Data
                </h2>
                <RangeCalendar
                    v-model="selectedRange"
                    class="rounded-md border"
                />
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import Papa from "papaparse";
import { ref, computed, watch, onMounted } from "vue";
import { RangeCalendar } from "~/components/ui/range-calendar";
import { parseDate } from "@internationalized/date";
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

// Registra os componentes do Chart.js
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

// 1) Dados importados (formato esperado):
// Cada objeto deve ter:
//   - data: string no formato parseável (ex.: "YYYY-MM-DD")
//   - valor: número (valor monitorado)
//   - valor_minimo: número (limite inferior)
//   - valor_maximo: número (limite superior)
//   - valor_alvo: número (valor de referência)
const cpkData = ref<
    Array<{
        data: string;
        valor: number;
        valor_minimo: number;
        valor_maximo: number;
        valor_alvo: number;
    }>
>([]);

// 2) Dados importados para exibição na tabela (CSV)
const parsedData = ref<Array<Record<string, string>>>([]);
const headers = ref<string[]>([]);

// 3) Range selecionado para o filtro
const start = parseDate("2025-02-01");
const end = parseDate("2025-02-10");
const selectedRange = ref<DateRange>({ start, end });

// 4) Computed que filtra os dados conforme o range selecionado
const getFilteredData = computed(() => {
    if (!selectedRange.value.start || !selectedRange.value.end) {
        console.log("Intervalo de datas não definido.");
        return [];
    }

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

    return cpkData.value.filter((d) => {
        const dataDate = new Date(d.data);
        return dataDate >= startDate && dataDate <= endDate;
    });
});

// 5) Estatísticas baseadas nos dados filtrados
const media = computed(() =>
    getFilteredData.value.length
        ? getFilteredData.value.reduce((sum, d) => sum + d.valor, 0) /
          getFilteredData.value.length
        : 0,
);

const desvioPadrao = computed(() => {
    if (!getFilteredData.value.length) return 0;
    const mean = media.value;
    const variancia =
        getFilteredData.value.reduce(
            (sum, d) => sum + Math.pow(d.valor - mean, 2),
            0,
        ) / getFilteredData.value.length;
    return Math.sqrt(variancia);
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

const cp = computed(() => {
    if (desvioPadrao.value === 0) return 0;
    return (maxValor.value - minValor.value) / (6 * desvioPadrao.value);
});

const cpk = computed(() => {
    if (desvioPadrao.value === 0) return 0;
    return Math.min(
        (media.value - minValor.value) / (3 * desvioPadrao.value),
        (maxValor.value - media.value) / (3 * desvioPadrao.value),
    );
});

// 6) Referência para o gráfico e instância do Chart.js
const chartRef = ref<HTMLCanvasElement | null>(null);
let chartInstance: Chart | null = null;

// 7) Função para atualizar o gráfico usando os dados filtrados
const updateChart = () => {
    if (!chartRef.value) return;

    if (!getFilteredData.value.length) {
        if (chartInstance) {
            chartInstance.destroy();
            chartInstance = null;
        }
        return;
    }

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
            scales: {
                y: {
                    min: minValor.value,
                    max: maxValor.value,
                },
            },
        },
    });
};

// Atualiza o gráfico sempre que o range ou os dados importados mudarem
watch(selectedRange, updateChart, { deep: true });
watch(cpkData, updateChart);

// Atualiza o gráfico ao montar o componente
onMounted(updateChart);

// 8) Função para processar o upload do CSV e converter os dados para o formato esperado
const handleFileUpload = (event: Event) => {
    const file = (event.target as HTMLInputElement).files?.[0];
    if (!file) return;

    Papa.parse(file, {
        header: true,
        skipEmptyLines: true,
        complete: (result) => {
            if (result.data.length > 0) {
                parsedData.value = result.data as Array<Record<string, string>>;
                headers.value = Object.keys(result.data[0]);

                // Converte cada linha do CSV para o formato esperado
                const convertedCSV = parsedData.value.map((row) => ({
                    data: row.data, // Ex.: "YYYY-MM-DD"
                    valor: parseFloat(row.valor),
                    valor_minimo: parseFloat(row.valor_minimo),
                    valor_maximo: parseFloat(row.valor_maximo),
                    valor_alvo: parseFloat(row.valor_alvo),
                }));

                // Atualiza os dados que serão filtrados e exibidos no gráfico
                cpkData.value = convertedCSV;
                updateChart();
            }
        },
        error: (error) => {
            console.error("Erro ao processar CSV:", error);
        },
    });
};
</script>
