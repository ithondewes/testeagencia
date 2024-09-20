<template>
<div class="grid">
    <!-- Card para a Análise de Arquivo -->
    <div class="col-12 xl:col-3">
        <div class="p-card">
            <div class="p-card-body">
                <div class="p-card-title">Análise de Arquivo</div>
                <div class="divider"></div>
                <div class="p-card-content">
                    <i class="pi pi-upload"></i>
                    <span class="upload-text">upload</span>
                </div>
            </div>
        </div>
    </div>
    <!-- Card para o Código de Barras -->
    <div class="col-12 xl:col-3">
        <div class="p-card">
            <div class="p-card-body">
                <div class="p-card-title">Código de Barras</div>
                <div class="divider"></div>
                <div class="p-card-content">
                    <i class="pi pi-upload"></i>
                    <span class="upload-text">upload</span>
                </div>
            </div>
        </div>
    </div>
    <!-- Card para o Cálculo de Tinta -->
    <div class="col-12 xl:col-3">
        <div class="p-card">
            <div class="p-card-body">
                <div class="p-card-title">Cálculo de Tinta</div>
                <div class="divider"></div>
                <div class="p-card-content">
                    <i class="pi pi-upload"></i>
                    <span class="upload-text">upload</span>
                </div>
            </div>
        </div>
    </div>
    <!-- Card vazio -->
    <div class="col-12 xl:col-3">
        <div class="p-card">
            <div class="p-card-body">
                <div class="p-card-title"><br></div>
                <div class="divider"></div>
                <div class="p-card-content">
                    <i class="pi pi-upload"></i>
                    <span class="upload-text">upload</span>
                </div>
            </div>
        </div>
    </div>
</div>

<div class="grid">
    <div class="col-12">
        <!-- DataTable para exibir as movimentações -->
        <div class="card">
            <DataTable 
                v-model:filters="filters" 
                :value="transactions" 
                paginator 
                :rows="10" 
                dataKey="os" 
                filterDisplay="row" 
                :loading="loading"
                :globalFilterFields="['os', 'entrega', 'cliente', 'titulo', 'fluxo', 'transporte', 'status']" 
                :rowHover="true" 
                v-model:selection="selectedOS" 
                selectionMode="checkbox">
                
                <!-- Cabeçalho da tabela -->
                <template #header>
                    <div class="flex justify-content-between flex-column sm:flex-row">
                        <div>
                            <h5> Movimentações </h5>
                            <h7 style="color: #808080;"> Lista das últimas movimentações </h7>
                        </div>
                        <span class="p-input-icon-left mb-2">
                            <i class="pi pi-search" />
                            <InputText v-model="filters['global'].value" placeholder="Pesquisar" style="width: 100%" />
                        </span>
                    </div>
                </template>

                <!-- Mensagem de tabela vazia -->
                <template #empty> Nenhuma movimentação encontrada. </template>
                
                <!-- Mensagem de carregamento -->
                <template #loading> Carregando dados das movimentações. Por favor, aguarde. </template>

                <!-- Coluna de seleção múltipla -->
                <Column selectionMode="multiple" headerStyle="width: 3em">
                    <template #header>
                        <div class="flex align-items-center">
                            <!-- Checkbox para selecionar/desmarcar todos os itens -->
                            <Checkbox 
                                v-model:checked="selectedOS" 
                                :indeterminate="selectedOS.length > 0 && selectedOS.length < transactions.length" 
                                @change="selectAll()" />
                            <span style="margin-left: 4px;"></span>
                        </div>
                    </template>
                </Column>

                <!-- Colunas da tabela com estilização dinâmica -->
                <Column field="os" header="OS" style="min-width: 8rem">
                    <template #body="{ data }">
                        <div :style="getRowStyle(data.fluxo)">
                            {{ data.os }}
                        </div>
                    </template>
                </Column>
                <Column field="entrega" header="Entrega" style="min-width: 10rem">
                    <template #body="{ data }">
                        <div :style="getRowStyle(data.fluxo)">
                            {{ data.entrega }}
                        </div>
                    </template>
                </Column>
                <Column field="cliente" header="Cliente" style="min-width: 10rem">
                    <template #body="{ data }">
                        <div :style="getRowStyle(data.fluxo)">
                            {{ data.cliente }}
                        </div>
                    </template>
                </Column>
                <Column field="titulo" header="Título" style="min-width: 20rem">
                    <template #body="{ data }">
                        <div :style="getRowStyle(data.fluxo)">
                            {{ data.titulo }}
                        </div>
                    </template>
                </Column>
                <Column field="fluxo" header="Fluxo" style="min-width: 12rem">
                    <template #body="{ data }">
                        <div :style="getRowStyle(data.fluxo)">
                            {{ data.fluxo }}
                        </div>
                    </template>
                </Column>
                <Column field="transporte" header="Transporte" style="min-width: 10rem">
                    <template #body="{ data }">
                        <div :style="getRowStyle(data.fluxo)">
                            {{ data.transporte }}
                        </div>
                    </template>
                </Column>
                <Column field="status" header="Status" style="min-width: 10rem">
                    <template #body="{ data }">
                        <div :style="getRowStyle(data.fluxo)">
                            {{ data.status }}
                        </div>
                    </template>
                </Column>
            </DataTable>
        </div>
    </div>
</div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import { useLayout } from '@/layout/composables/layout';
import { FilterMatchMode } from 'primevue/api';
import ProductService from '@/service/ProductService';
import Checkbox from 'primevue/checkbox';

const { isDarkTheme } = useLayout(); // Estado de tema escuro/claro do layout

const products = ref(null); // Dados dos produtos
const clienteCount = ref(null); // Contagem de clientes
const patrimonioCustodia = ref(null); // Patrimônio em custódia
const patrimonioMedio = ref(null); // Patrimônio médio
const equityHistory = ref([]); // Histórico de patrimônio
const transactions = ref([]); // Transações/movimentações
const selectedOS = ref([]); // Itens selecionados na tabela
const filters = ref({
  global: { value: null, matchMode: FilterMatchMode.CONTAINS } // Filtro global para busca
});
const loading = ref(true); // Indica se os dados estão sendo carregados

const productService = new ProductService(); // Serviço para buscar produtos

// Função para formatar a data para o padrão brasileiro
const formatDate = (value) => {
  return value.toLocaleDateString('pt-BR', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric'
  });
};

// Função para formatar valores monetários para o formato BRL
const formatCurrency = (value) => {
    return value.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
};

// Dados fictícios para preencher a tabela
const transactionsData = [
  { os: '145582', entrega: '14/05/2019', cliente: 'Flexcor', titulo: 'Flexograv - Teste Sistema', fluxo: 'Conferência', transporte: 'Motoboy', status: 'Conferência' },
  { os: '145583', entrega: '15/05/2019', cliente: 'Companhia ABC', titulo: 'Design Gráfico - Sistema XYZ', fluxo: 'Aguardando', transporte: 'Motoboy', status: 'Aguardando' },
  { os: '145584', entrega: '16/05/2019', cliente: 'Empresa Testes', titulo: 'Projeto de Desenvolvimento', fluxo: 'Gravação', transporte: 'Motoboy', status: 'Gravação' },
  { os: '145585', entrega: '17/05/2019', cliente: 'Tech Solutions', titulo: 'Análise de Software', fluxo: 'Gravação', transporte: 'Motoboy', status: 'Gravação' },
  { os: '145586', entrega: '18/05/2019', cliente: 'Flexcorp', titulo: 'Implementação de Sistema', fluxo: 'Aguardando', transporte: 'Motoboy', status: 'Aguardando' },
  { os: '145587', entrega: '19/05/2019', cliente: 'Digital Media', titulo: 'Criação de Website', fluxo: 'Financeiro', transporte: 'Motoboy', status: 'Financeiro' },
];

// Função executada quando o componente é montado
onMounted(() => {
    // Preenchendo a tabela com dados fictícios
    transactions.value = transactionsData;

    // Buscando produtos do serviço (assíncrono)
    productService.getProductsSmall().then((data) => (products.value = data));
    loading.value = false; // Indicando que o carregamento terminou
});

// Função para aplicar estilos dinâmicos a cada tipo de fluxo
const getRowStyle = (fluxo) => {
  switch (fluxo) {
    case 'Conferência':
      return { backgroundColor: '#FFE699', color: '#4D4D4D', height: '20px', lineHeight: '20px' };
    case 'Aguardando':
      return { backgroundColor: '#99FFCC', color: '#4D4D4D', height: '20px', lineHeight: '20px' };
    case 'Gravação':
      return { backgroundColor: '#FFCCCC', color: '#4D4D4D', height: '20px', lineHeight: '20px' };
    case 'Financeiro':
      return { backgroundColor: '#ADD8E6', color: '#4D4D4D', height: '20px', lineHeight: '20px' };
    default:
      return { backgroundColor: 'white', color: '#4D4D4D', height: '20px', lineHeight: '20px' };
  }
};

// Função para selecionar todos os itens da tabela
const selectAll = () => {
  if (selectedOS.value.length === transactions.value.length) {
    selectedOS.value = [];
  } else {
    selectedOS.value = [...transactions.value];
  }
};
</script>

<style scoped>
.p-datatable .p-datatable-tbody > tr > td {
    padding: 0 !important; /* Remove o padding das células */
}

.p-datatable .p-datatable-tbody > tr > td > div {
    padding: 0 !important; /* Remove o padding dos divs internos */
}
</style>
