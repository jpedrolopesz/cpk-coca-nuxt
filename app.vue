<template>
    <SidebarProvider>
        <!-- Layout com Sidebar e Conteúdo Principal -->
        <div class="flex h-screen w-full">
            <!-- SIDEBAR -->
            <Sidebar class="w-64">
                <SidebarHeader>
                    <SidebarMenu>
                        <SidebarMenuItem>
                            <DropdownMenu>
                                <DropdownMenuTrigger as-child>
                                    <SidebarMenuButton
                                        size="lg"
                                        :class="{
                                            'data-[state=open]:bg-sidebar-accent data-[state=open]:text-sidebar-accent-foreground':
                                                dropdownOpen,
                                        }"
                                        @click="toggleDropdown"
                                    >
                                        <div
                                            class="flex aspect-square size-8 items-center justify-center rounded-lg bg-sidebar-primary text-sidebar-primary-foreground"
                                        >
                                            <GalleryVerticalEnd
                                                class="size-4"
                                            />
                                        </div>
                                        <div
                                            class="flex flex-col gap-0.5 leading-none"
                                        >
                                            <span class="font-semibold"
                                                >COCA COLA FEMSA</span
                                            >
                                            <span>{{ selectedVersion }}</span>
                                        </div>
                                        <ChevronsUpDown class="ml-auto" />
                                    </SidebarMenuButton>
                                </DropdownMenuTrigger>
                                <DropdownMenuContent
                                    v-if="dropdownOpen"
                                    class="w-[--radix-dropdown-menu-trigger-width]"
                                    align="start"
                                >
                                    <DropdownMenuItem
                                        v-for="version in data.versions"
                                        :key="version"
                                        @click="setSelectedVersion(version)"
                                    >
                                        {{ version }}
                                        <Check
                                            v-if="version === selectedVersion"
                                            class="ml-auto"
                                        />
                                    </DropdownMenuItem>
                                </DropdownMenuContent>
                            </DropdownMenu>
                        </SidebarMenuItem>
                    </SidebarMenu>

                    <!-- Formulário de Busca -->
                    <form @submit.prevent>
                        <SidebarGroup class="py-0">
                            <SidebarGroupContent class="relative">
                                <Label for="search" class="sr-only"
                                    >Search</Label
                                >
                                <SidebarInput
                                    id="search"
                                    v-model="search"
                                    placeholder="Search the docs..."
                                    class="pl-8"
                                />
                                <Search
                                    class="pointer-events-none absolute left-2 top-1/2 size-4 -translate-y-1/2 select-none opacity-50"
                                />
                            </SidebarGroupContent>
                        </SidebarGroup>
                    </form>
                </SidebarHeader>

                <!-- MENU DE NAVEGAÇÃO -->
                <SidebarContent>
                    <SidebarGroup
                        v-for="item in data.navMain"
                        :key="item.title"
                    >
                        <SidebarGroupLabel>{{ item.title }}</SidebarGroupLabel>
                        <SidebarGroupContent>
                            <SidebarMenu>
                                <SidebarMenuItem
                                    v-for="subItem in item.items"
                                    :key="subItem.title"
                                >
                                    <SidebarMenuButton
                                        :class="{
                                            'is-active': subItem.isActive,
                                        }"
                                        as-child
                                    >
                                        <a :href="subItem.url">{{
                                            subItem.title
                                        }}</a>
                                    </SidebarMenuButton>
                                </SidebarMenuItem>
                            </SidebarMenu>
                        </SidebarGroupContent>
                    </SidebarGroup>
                </SidebarContent>

                <SidebarRail />
            </Sidebar>

            <!-- CONTEÚDO PRINCIPAL DO DASHBOARD -->
            <SidebarInset class="flex-1 w-full p-0 m-0">
                <!-- Container Flex vertical sem padding para ocupar a largura total -->
                <div class="flex flex-col h-full w-full">
                    <!-- Cabeçalho com Breadcrumb -->
                    <header
                        class="flex justify-center h-16 shrink-0 items-center gap-2 border-b px-2"
                    >
                        <div class="flex space-x-6">
                            <SidebarTrigger class="ml-4" />
                            <h2
                                class="text-xl font-bold text-gray-800 mb-4 text-center"
                            >
                                Monitoramento de CPK e Tendências
                            </h2>
                        </div>

                        <!-- Container com botões de ação -->
                        <div class="flex items-center space-x-4 p-2 ml-auto">
                            <!-- Botão para Upload de CSV -->
                            <input
                                ref="fileInputRef"
                                type="file"
                                @change="handleFileUpload"
                                accept=".csv"
                                class="hidden"
                            />
                            <Button @click="triggerFileInput">
                                Upload CVS
                            </Button>

                            <!-- Botão para Gerar Relatório -->
                            <Button @click="generateReport">
                                Gerar Relatório
                            </Button>

                            <!-- Botão para abrir o Drawer com dados importados -->
                            <Drawer>
                                <DrawerTrigger>
                                    <button
                                        class="p-1 bg-gray-500 hover:bg-gray-600 rounded-full text-white transition-colors shadow-md"
                                    >
                                        <EyeIcon />
                                    </button>
                                </DrawerTrigger>
                                <DrawerContent class="h-full">
                                    <div v-if="parsedData.length" class="p-2">
                                        <h3
                                            class="text-lg font-semibold text-gray-700 mb-2"
                                        >
                                            Dados Importados
                                        </h3>
                                        <div
                                            class="overflow-auto h-screen border border-gray-200 rounded-md p-2"
                                        >
                                            <table
                                                class="w-full text-sm text-left"
                                            >
                                                <thead class="bg-gray-100">
                                                    <tr>
                                                        <th
                                                            v-for="(
                                                                header, index
                                                            ) in headers"
                                                            :key="index"
                                                            class="border p-2"
                                                        >
                                                            {{ header }}
                                                        </th>
                                                    </tr>
                                                </thead>
                                                <tbody>
                                                    <tr
                                                        v-for="(
                                                            row, rowIndex
                                                        ) in parsedData"
                                                        :key="rowIndex"
                                                        class="border hover:bg-gray-50"
                                                    >
                                                        <td
                                                            v-for="(
                                                                value, key
                                                            ) in row"
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
                                    <div
                                        v-else
                                        class="p-2 text-center text-gray-500"
                                    >
                                        Nenhum dado importado.
                                    </div>
                                </DrawerContent>
                            </Drawer>

                            <!-- POPUP para seleção de intervalo de datas -->
                            <Popover>
                                <PopoverTrigger as-child>
                                    <Button
                                        id="date"
                                        variant="outline"
                                        class="w-[300px] justify-start text-left font-normal"
                                    >
                                        <CalendarIcon class="mr-2 h-4 w-4" />
                                        <template v-if="selectedRange.start">
                                            <template v-if="selectedRange.end">
                                                {{
                                                    df.format(
                                                        selectedRange.start.toDate(
                                                            getLocalTimeZone(),
                                                        ),
                                                    )
                                                }}
                                                -
                                                {{
                                                    df.format(
                                                        selectedRange.end.toDate(
                                                            getLocalTimeZone(),
                                                        ),
                                                    )
                                                }}
                                            </template>
                                            <template v-else>
                                                {{
                                                    df.format(
                                                        selectedRange.start.toDate(
                                                            getLocalTimeZone(),
                                                        ),
                                                    )
                                                }}
                                            </template>
                                        </template>
                                        <template v-else>
                                            Pick a date
                                        </template>
                                    </Button>
                                </PopoverTrigger>
                                <PopoverContent class="w-auto p-0" align="end">
                                    <RangeCalendar
                                        v-model="selectedRange"
                                        weekday-format="short"
                                        :number-of-months="2"
                                        initial-focus
                                        :placeholder="selectedRange.start"
                                        @update:start-value="
                                            (startDate) =>
                                                (selectedRange.start =
                                                    startDate)
                                        "
                                    />
                                </PopoverContent>
                            </Popover>
                        </div>
                    </header>

                    <!-- Área de Conteúdo do Dashboard (Gráfico e Estatísticas) ocupando 100% da largura -->
                    <div class="flex-1 p-8">
                        <div
                            class="bg-white border border-gray-300 shadow-lg rounded-lg w-full h-auto flex flex-col"
                        >
                            <div
                                class="flex-1 flex justify-center items-center"
                            >
                                <canvas
                                    ref="chartRef"
                                    class="w-full h-56"
                                ></canvas>
                            </div>
                            <div class="p-4">
                                <!-- Estatísticas Gerais (todos os materiais) -->
                                <h2
                                    class="text-lg font-semibold text-gray-700 mb-3 border-b pb-2"
                                >
                                    Estatísticas (Geral)
                                </h2>
                                <ul
                                    class="space-y-3 text-gray-600 text-sm border border-gray-200 rounded p-3"
                                >
                                    <li>Média: {{ media.toFixed(2) }}</li>
                                    <li>
                                        Desvio Padrão:
                                        {{ desvioPadrao.toFixed(2) }}
                                    </li>
                                    <li>CPK: {{ cpk.toFixed(2) }}</li>
                                    <li>CP: {{ cp.toFixed(2) }}</li>
                                    <li>
                                        Amplitude (Máx - Mín):
                                        {{ amplitude.toFixed(2) }}
                                    </li>
                                </ul>

                                <!-- Estatísticas por Material -->
                                <h2
                                    class="text-lg font-semibold text-gray-700 mb-3 mt-8 border-b pb-2"
                                >
                                    Estatísticas (Por Material)
                                </h2>
                                <!-- Se existir pelo menos um material, exibimos abaixo -->
                                <div
                                    v-if="statsByMaterialComputed.length"
                                    class="space-y-4"
                                >
                                    <div
                                        v-for="(
                                            item, idx
                                        ) in statsByMaterialComputed"
                                        :key="idx"
                                        class="border border-gray-200 rounded p-3"
                                    >
                                        <h3
                                            class="font-semibold text-gray-700 mb-2"
                                        >
                                            Material: {{ item.material }}
                                        </h3>
                                        <ul
                                            class="space-y-1 text-gray-600 text-sm"
                                        >
                                            <li>Média: {{ item.media }}</li>
                                            <li>
                                                Desvio Padrão: {{ item.desvio }}
                                            </li>
                                            <li>CP: {{ item.cp }}</li>
                                            <li>CPK: {{ item.cpk }}</li>
                                            <li>
                                                Amplitude: {{ item.amplitude }}
                                            </li>
                                        </ul>
                                    </div>
                                </div>
                                <div v-else class="mt-2 text-gray-500 text-sm">
                                    Nenhum dado por material.
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </SidebarInset>
        </div>
    </SidebarProvider>
</template>

<script setup lang="ts">
// IMPORTS DE COMPONENTES DE POPUP E BOTÃO
import {
    Popover,
    PopoverContent,
    PopoverTrigger,
} from "~/components/ui/popover";
import { Button } from "~/components/ui/button";
import {
    Calendar as CalendarIcon,
    UploadIcon,
    FileTextIcon,
    EyeIcon,
    Check,
    ChevronsUpDown,
    GalleryVerticalEnd,
    Search,
} from "lucide-vue-next";

// IMPORTS DO INTERNATIONALIZED DATE
import {
    DateFormatter,
    getLocalTimeZone,
    parseDate,
} from "@internationalized/date";

// IMPORTS DO VUE
import { ref, computed, watch, onMounted } from "vue";

// IMPORTS PARA CSV e CHART.JS
import Papa from "papaparse";
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

// IMPORTS DOS COMPONENTES DO DRAWER
import { Drawer, DrawerTrigger, DrawerContent } from "~/components/ui/drawer";

// IMPORTS DOS COMPONENTES DE SIDEBAR E NAVIGAÇÃO
import {
    Sidebar,
    SidebarContent,
    SidebarGroup,
    SidebarGroupContent,
    SidebarGroupLabel,
    SidebarHeader,
    SidebarInput,
    SidebarInset,
    SidebarMenu,
    SidebarMenuButton,
    SidebarMenuItem,
    SidebarProvider,
    SidebarRail,
    SidebarTrigger,
} from "~/components/ui/sidebar";

// IMPORTS DO DROPDOWN
import {
    DropdownMenu,
    DropdownMenuContent,
    DropdownMenuItem,
    DropdownMenuTrigger,
} from "~/components/ui/dropdown-menu";

// IMPORTS DE LABEL, SEPARATOR E BREADCRUMB
import { Label } from "~/components/ui/label";
import { Separator } from "~/components/ui/separator";
import {
    Breadcrumb,
    BreadcrumbItem,
    BreadcrumbLink,
    BreadcrumbList,
    BreadcrumbPage,
    BreadcrumbSeparator,
} from "~/components/ui/breadcrumb";

// IMPORTAÇÃO DO RANGE CALENDAR (usado tanto no popover quanto para filtragem)
import { RangeCalendar } from "~/components/ui/range-calendar";

// CRIAÇÃO DE UM DATE FORMATTER para formatar as datas selecionadas
const df = new DateFormatter("en-US", {
    dateStyle: "medium",
});

// ----- ESTADO E LÓGICA DA SIDEBAR E DASHBOARD -----
const data = {
    versions: ["G3", "G2", "G1"],
    navMain: [
        {
            title: "Del Valle Kapo",
            url: "#",
            items: [
                { title: "Concentraçao Peróxido", url: "#" },
                { title: "Conteúdo Líquido", url: "#" },
                { title: "Brix", url: "#" },
                { title: "Acidez", url: "#" },
                { title: "pH", url: "#" },
            ],
        },
    ],
};

const fileInputRef = ref<HTMLInputElement | null>(null);

const triggerFileInput = () => {
    fileInputRef.value?.click();
};

const selectedVersion = ref(data.versions[0]);
const dropdownOpen = ref(false);
const search = ref("");

function toggleDropdown() {
    dropdownOpen.value = !dropdownOpen.value;
}
function setSelectedVersion(version: string) {
    selectedVersion.value = version;
}

// ----- ESTADO PARA CSV, CHART E FILTRAGEM POR DATA -----
// Incluindo o campo "material"
const cpkData = ref<
    Array<{
        data: string;
        valor: number;
        valor_minimo: number;
        valor_maximo: number;
        valor_alvo: number;
        material: string;
    }>
>([]);

const parsedData = ref<Array<Record<string, string>>>([]);
const headers = ref<string[]>([]);

// Definindo o range inicial
const start = parseDate("2025-02-01");
const end = parseDate("2025-02-10");
const selectedRange = ref({ start, end });

// Filtra os dados com base no intervalo selecionado
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

// ----- AGRUPAMENTO POR PRODUTO -----
const groupedData = computed(() => {
    return getFilteredData.value.reduce(
        (groups, row) => {
            const key = row.material;
            if (!groups[key]) {
                groups[key] = [];
            }
            groups[key].push(row);
            return groups;
        },
        {} as Record<string, typeof getFilteredData.value>,
    );
});

// ----- CÁLCULOS ESTATÍSTICOS (GERAIS) -----
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

// ----- FUNÇÃO AUXILIAR PARA CÁLCULO DE ESTATÍSTICAS POR MATERIAL -----
function computeStats(records: typeof cpkData.value) {
    if (!records.length) {
        return { media: 0, desvio: 0, cp: 0, cpk: 0, amplitude: 0 };
    }
    const mean = records.reduce((acc, r) => acc + r.valor, 0) / records.length;
    const variance =
        records.reduce((acc, r) => acc + Math.pow(r.valor - mean, 2), 0) /
        records.length;
    const stdDev = Math.sqrt(variance);
    const minVal = Math.min(...records.map((r) => r.valor_minimo));
    const maxVal = Math.max(...records.map((r) => r.valor_maximo));
    const amplitudeVal = maxVal - minVal;
    const cpVal = stdDev === 0 ? 0 : (maxVal - minVal) / (6 * stdDev);
    const cpkVal =
        stdDev === 0
            ? 0
            : Math.min(
                  (mean - minVal) / (3 * stdDev),
                  (maxVal - mean) / (3 * stdDev),
              );

    return {
        media: mean,
        desvio: stdDev,
        cp: cpVal,
        cpk: cpkVal,
        amplitude: amplitudeVal,
    };
}

// ----- ESTATÍSTICAS POR MATERIAL (COMPUTED) -----
const statsByMaterialComputed = computed(() => {
    return Object.entries(groupedData.value).map(([material, records]) => {
        const stats = computeStats(records);
        return {
            material,
            media: stats.media.toFixed(2),
            desvio: stats.desvio.toFixed(2),
            cp: stats.cp.toFixed(2),
            cpk: stats.cpk.toFixed(2),
            amplitude: stats.amplitude.toFixed(2),
        };
    });
});

// ----- CONFIGURAÇÃO DO CHART.JS -----
const chartRef = ref<HTMLCanvasElement | null>(null);
let chartInstance: Chart | null = null;

const updateChart = () => {
    if (!chartRef.value) return;
    if (!getFilteredData.value.length) {
        if (chartInstance) {
            chartInstance.destroy();
            chartInstance = null;
        }
        return;
    }
    // Cria um conjunto unificado de labels (datas) a partir dos dados filtrados
    const labelsSet = new Set(
        getFilteredData.value.map((d) =>
            new Date(d.data).toLocaleDateString("pt-BR"),
        ),
    );
    const labels = Array.from(labelsSet).sort(
        (a, b) => new Date(a).getTime() - new Date(b).getTime(),
    );

    // Array de cores para diferenciar os produtos
    const colors = [
        "blue",
        "red",
        "green",
        "orange",
        "purple",
        "teal",
        "brown",
    ];
    const datasets: any[] = [];

    // Para cada grupo (produto), gera 4 datasets: Monitorados, CPK Alvo, Valor Mínimo e Valor Máximo
    Object.entries(groupedData.value).forEach(([material, records], index) => {
        const baseColor = colors[index % colors.length];
        const monitored = labels.map((label) => {
            const rec = records.find(
                (r) => new Date(r.data).toLocaleDateString("pt-BR") === label,
            );
            return rec ? rec.valor : null;
        });
        const alvo = labels.map((label) => {
            const rec = records.find(
                (r) => new Date(r.data).toLocaleDateString("pt-BR") === label,
            );
            return rec ? rec.valor_alvo : null;
        });
        const minimo = labels.map((label) => {
            const rec = records.find(
                (r) => new Date(r.data).toLocaleDateString("pt-BR") === label,
            );
            return rec ? rec.valor_minimo : null;
        });
        const maximo = labels.map((label) => {
            const rec = records.find(
                (r) => new Date(r.data).toLocaleDateString("pt-BR") === label,
            );
            return rec ? rec.valor_maximo : null;
        });

        datasets.push({
            label: `Produto ${material} - Valores Monitorados`,
            data: monitored,
            borderColor: baseColor,
            borderWidth: 2,
            fill: false,
        });
        datasets.push({
            label: `Produto ${material} - CPK Alvo`,
            data: alvo,
            borderColor: baseColor,
            borderDash: [5, 5],
            pointRadius: 0,
            fill: false,
        });
        datasets.push({
            label: `Produto ${material} - Valor Mínimo`,
            data: minimo,
            borderColor: baseColor,
            borderDash: [5, 5],
            pointRadius: 0,
            fill: false,
        });
        datasets.push({
            label: `Produto ${material} - Valor Máximo`,
            data: maximo,
            borderColor: baseColor,
            borderDash: [5, 5],
            pointRadius: 0,
            fill: false,
        });
    });

    if (chartInstance) chartInstance.destroy();
    chartInstance = new Chart(chartRef.value, {
        type: "line",
        data: {
            labels,
            datasets,
        },
        options: {
            responsive: true,
            scales: {
                y: { min: minValor.value, max: maxValor.value },
            },
        },
    });
};

watch(selectedRange, updateChart, { deep: true });
watch(cpkData, updateChart);
onMounted(updateChart);

// ----- PROCESSAMENTO DO UPLOAD DO CSV -----
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
                const convertedCSV = parsedData.value.map((row) => ({
                    data: row.data,
                    valor: parseFloat(row.valor),
                    valor_minimo: parseFloat(row.valor_minimo),
                    valor_maximo: parseFloat(row.valor_maximo),
                    valor_alvo: parseFloat(row.valor_alvo),
                    material: row.material || "", // Incluindo o novo campo "material"
                }));
                cpkData.value = convertedCSV;
                updateChart();
            }
        },
        error: (error) => {
            console.error("Erro ao processar CSV:", error);
        },
    });
};

// ----- GERAÇÃO DE RELATÓRIO (OPCIONAL) -----
const generateReport = () => {
    // Formata as datas de início e fim, se existirem
    const startString = selectedRange.value.start
        ? df.format(selectedRange.value.start.toDate(getLocalTimeZone()))
        : "N/A";
    const endString = selectedRange.value.end
        ? df.format(selectedRange.value.end.toDate(getLocalTimeZone()))
        : "N/A";

    // Estatísticas gerais (já calculadas nas computed properties)
    const generalStats = {
        media: media.value.toFixed(2),
        desvio: desvioPadrao.value.toFixed(2),
        cp: cp.value.toFixed(2),
        cpk: cpk.value.toFixed(2),
        amplitude: amplitude.value.toFixed(2),
    };

    // Estatísticas por material, usando sua função auxiliar ou computed
    const statsByMaterial = statsByMaterialComputed.value;
    // (Supondo que você já tenha a computed property statsByMaterialComputed)

    // Gera HTML para exibir estatísticas por material
    let statsByMaterialHTML = `
    <h3>Estatísticas por Material</h3>
    <table border="1" cellspacing="0" cellpadding="5" style="margin-top:10px; width:100%;">
      <tr>
        <th>Material</th>
        <th>Média</th>
        <th>Desvio Padrão</th>
        <th>CP</th>
        <th>CPK</th>
        <th>Amplitude</th>
      </tr>
  `;
    statsByMaterial.forEach((item) => {
        statsByMaterialHTML += `
      <tr>
        <td>${item.material}</td>
        <td>${item.media}</td>
        <td>${item.desvio}</td>
        <td>${item.cp}</td>
        <td>${item.cpk}</td>
        <td>${item.amplitude}</td>
      </tr>
    `;
    });
    statsByMaterialHTML += "</table>";

    // Se quiser incluir o gráfico como imagem
    let imageData = "";
    if (chartRef.value) {
        imageData = chartRef.value.toDataURL("image/png");
    }

    // Monta o HTML final do relatório
    const reportHTML = `
    <html>
      <head>
        <title>Relatório de Monitoramento</title>
        <style>
          body { font-family: Arial, sans-serif; margin: 20px; }
          h1, h2, h3 { color: #333; }
          table { border-collapse: collapse; margin-top: 10px; }
          th, td { border: 1px solid #ccc; padding: 8px; text-align: center; }
          .stats { margin: 20px 0; }
          .chart { margin-top: 20px; }
        </style>
      </head>
      <body>
        <h1>Relatório de Monitoramento</h1>
        <p><strong>Data do Relatório:</strong> ${new Date().toLocaleString("pt-BR")}</p>
        <p><strong>Período Selecionado:</strong> ${startString} - ${endString}</p>

        <div class="stats">
          <h2>Estatísticas Gerais</h2>
          <ul>
            <li><strong>Média:</strong> ${generalStats.media}</li>
            <li><strong>Desvio Padrão:</strong> ${generalStats.desvio}</li>
            <li><strong>CP:</strong> ${generalStats.cp}</li>
            <li><strong>CPK:</strong> ${generalStats.cpk}</li>
            <li><strong>Amplitude:</strong> ${generalStats.amplitude}</li>
          </ul>
        </div>

        <div class="stats">
          ${statsByMaterial.length > 0 ? statsByMaterialHTML : "<p>Nenhum dado por material.</p>"}
        </div>

        <div class="chart">
          <h2>Gráfico</h2>
          ${
              imageData
                  ? `<img src="${imageData}" alt="Gráfico de Monitoramento" style="max-width: 100%; border:1px solid #ccc;">`
                  : "<p>Gráfico não disponível.</p>"
          }
        </div>
      </body>
    </html>
  `;

    // Abre em nova aba (ou janela) para exibir o relatório
    const reportWindow = window.open("", "_blank");
    if (reportWindow) {
        reportWindow.document.write(reportHTML);
        reportWindow.document.close();
    } else {
        console.error("Falha ao abrir a janela do relatório.");
    }
};

// REGISTRO DOS COMPONENTES DO CHART.JS
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
</script>
