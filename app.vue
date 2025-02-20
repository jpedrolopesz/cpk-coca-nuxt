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
                            <button
                                @click="triggerFileInput"
                                class="p-1 bg-blue-500 hover:bg-blue-600 rounded-full text-white transition-colors shadow-md"
                            >
                                <UploadIcon />
                            </button>

                            <!-- Botão para Gerar Relatório -->
                            <button
                                @click="generateReport"
                                class="p-1 bg-green-500 hover:bg-green-600 rounded-full text-white transition-colors shadow-md"
                            >
                                <FileTextIcon />
                            </button>

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
                                        <span class="font-medium"
                                            >Desvio Padrão (σ):</span
                                        >
                                        <span>{{
                                            desvioPadrao.toFixed(2)
                                        }}</span>
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
                                        <span class="font-medium"
                                            >Amplitude (Máx - Mín):</span
                                        >
                                        <span>{{ amplitude.toFixed(2) }}</span>
                                    </li>
                                </ul>
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
    versions: ["G1", "G2", "G3"],
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
const cpkData = ref<
    Array<{
        data: string;
        valor: number;
        valor_minimo: number;
        valor_maximo: number;
        valor_alvo: number;
    }>
>([]);

const parsedData = ref<Array<Record<string, string>>>([]);
const headers = ref<string[]>([]);

// Usamos parseDate para definir o range inicial (ajuste conforme sua necessidade)
const start = parseDate("2025-02-01");
const end = parseDate("2025-02-10");
const selectedRange = ref({ start, end });

// Filtra os dados conforme o intervalo selecionado
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

// Cálculo das estatísticas
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

// CONFIGURAÇÃO DO CHART.JS
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
                y: { min: minValor.value, max: maxValor.value },
            },
        },
    });
};

watch(selectedRange, updateChart, { deep: true });
watch(cpkData, updateChart);
onMounted(updateChart);

// PROCESSAMENTO DO UPLOAD DO CSV
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

const generateReport = () => {
    const reportData = {
        dataRelatorio: new Date().toLocaleString("pt-BR"),
        media: media.value.toFixed(2),
        desvio: desvioPadrao.value.toFixed(2),
        cp: cp.value.toFixed(2),
        cpk: cpk.value.toFixed(2),
        amplitude: amplitude.value.toFixed(2),
    };
    let imageData = "";
    if (chartRef.value) {
        imageData = chartRef.value.toDataURL("image/png");
    }
    const reportHTML = `
    <html>
      <head>
        <title>Relatório de Monitoramento</title>
        <style>
          body { font-family: Arial, sans-serif; margin: 20px; }
          h1 { color: #333; }
          .stats { margin: 20px 0; }
          .stats li { margin-bottom: 5px; }
          .chart { margin-top: 20px; }
        </style>
      </head>
      <body>
        <h1>Relatório de Monitoramento</h1>
        <p><strong>Data do Relatório:</strong> ${reportData.dataRelatorio}</p>
        <div class="stats">
          <h2>Estatísticas</h2>
          <ul>
            <li><strong>Média:</strong> ${reportData.media}</li>
            <li><strong>Desvio Padrão:</strong> ${reportData.desvio}</li>
            <li><strong>CP:</strong> ${reportData.cp}</li>
            <li><strong>CPK:</strong> ${reportData.cpk}</li>
            <li><strong>Amplitude:</strong> ${reportData.amplitude}</li>
          </ul>
        </div>
        <div class="chart">
          <h2>Gráfico</h2>
          ${imageData ? `<img src="${imageData}" alt="Gráfico de Monitoramento" style="max-width: 100%;">` : "<p>Gráfico não disponível.</p>"}
        </div>
      </body>
    </html>
  `;
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
