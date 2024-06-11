<script setup>
import { FilterMatchMode } from 'primevue/api';
import { ref, onMounted, onBeforeMount } from 'vue';
import ProductService from '@/service/ProductService';
import { useToast } from 'primevue/usetoast';
import axios from 'axios';

const toast = useToast();

const products = ref([]); // Initialize as an empty array
const productDialog = ref(false);
const deleteProductDialog = ref(false);
const deleteProductsDialog = ref(false);
const product = ref({});
const selectedProducts = ref(null);
const dt = ref(null);
const filters = ref({});
const submitted = ref(false);
const statuses = ref([
    { label: 'INSTOCK', value: 'instock' },
    { label: 'LOWSTOCK', value: 'lowstock' },
    { label: 'OUTOFSTOCK', value: 'outofstock' }
]);

onBeforeMount(() => {
    initFilters();
});

onMounted(() => {
    // Add some fictitious data
    products.value = [
        { id: '1', razaoSocial: 'Company A', cnpj: '00.000.000/0000-00', telefone: '(51) 1111-1111', email: 'companya@example.com' },
        { id: '2', razaoSocial: 'Company B', cnpj: '11.111.111/1111-11', telefone: '(51) 2222-2222', email: 'companyb@example.com' },
        { id: '3', razaoSocial: 'Company C', cnpj: '22.222.222/2222-22', telefone: '(51) 3333-3333', email: 'companyc@example.com' },
    ];
});

const formatCurrency = (value) => {
    return value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
};

const openNew = () => {
    product.value = {};
    submitted.value = false;
    productDialog.value = true;
};

const hideDialog = () => {
    productDialog.value = false;
    submitted.value = false;
};

const saveProduct = () => {
    submitted.value = true;
    if (product.value.razaoSocial && product.value.razaoSocial.trim() && product.value.cnpj) {
        if (product.value.id) {
            products.value[findIndexById(product.value.id)] = product.value;
            toast.add({ severity: 'success', summary: 'Successful', detail: 'Company Updated', life: 3000 });
        } else {
            product.value.id = createId();
            product.value.code = createId();
            products.value.push(product.value);
            toast.add({ severity: 'success', summary: 'Successful', detail: 'Company Created', life: 3000 });
        }
        productDialog.value = false;
        product.value = {};
    }
};

const editProduct = (editProduct) => {
    product.value = { ...editProduct };
    console.log(product);
    productDialog.value = true;
};

const confirmDeleteProduct = (editProduct) => {
    product.value = editProduct;
    deleteProductDialog.value = true;
};

const deleteProduct = () => {
    products.value = products.value.filter((val) => val.id !== product.value.id);
    deleteProductDialog.value = false;
    product.value = {};
    toast.add({ severity: 'success', summary: 'Successful', detail: 'Company Deleted', life: 3000 });
};

const findIndexById = (id) => {
    let index = -1;
    for (let i = 0; i < products.value.length; i++) {
        if (products.value[i].id === id) {
            index = i;
            break;
        }
    }
    return index;
};

const createId = () => {
    let id = '';
    const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    for (let i = 0; i < 5; i++) {
        id += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return id;
};

const exportCSV = () => {
    dt.value.exportCSV();
};

const confirmDeleteSelected = () => {
    deleteProductsDialog.value = true;
};
const deleteSelectedProducts = () => {
    products.value = products.value.filter((val) => !selectedProducts.value.includes(val));
    deleteProductsDialog.value = false;
    selectedProducts.value = null;
    toast.add({ severity: 'success', summary: 'Successful', detail: 'Companies Deleted', life: 3000 });
};

const initFilters = () => {
    filters.value = {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS }
    };
};

const fetchAddressByPostalCode = async (cep) => {
    try {
        const response = await axios.get(`https://viacep.com.br/ws/${cep}/json/`);
        if (response.data.erro) {
            toast.add({ severity: 'error', summary: 'Error', detail: 'CEP não encontrado', life: 3000 });
            product.value.rua = '';
            product.value.bairro = '';
            product.value.cidade = '';
        } else {
            const address = response.data;
            product.value.rua = address.logradouro;
            product.value.bairro = address.bairro;
            product.value.cidade = address.localidade;
        }
    } catch (error) {
        toast.add({ severity: 'error', summary: 'Error', detail: 'Erro ao buscar endereço', life: 3000 });
    }
};

const validateCNPJ = () => {
    let cnpj = product.value.cnpj.replace(/\D/g, '');
    if (cnpj.length === 14) {
        cnpj = cnpj.replace(/^(\d{2})(\d{3})(\d{3})(\d{4})(\d{2})$/, '$1.$2.$3/$4-$5');
        product.value.cnpj = cnpj;
    } else {
        toast.add({ severity: 'error', summary: 'Error', detail: 'CNPJ inválido', life: 3000 });
        product.value.cnpj = '';
    }
};

const validateTelefone = () => {
    let telefone = product.value.telefone.replace(/\D/g, '');
    if (telefone.length === 10 || telefone.length === 11) {
        if (telefone.length === 11) {
            telefone = telefone.replace(/^(\d{2})(\d{5})(\d{4})$/, '($1) $2-$3');
        } else {
            telefone = telefone.replace(/^(\d{2})(\d{4})(\d{4})$/, '($1) $2-$3');
        }
        product.value.telefone = telefone;
    } else {
        toast.add({ severity: 'error', summary: 'Error', detail: 'Telefone inválido', life: 3000 });
        product.value.telefone = '';
    }
};


</script>

<template>
    <div class="grid">
        <div class="col-12">
            <div class="card">
                <Toast />
                <Toolbar class="mb-4">
                    <template v-slot:start>
                        <div class="my-2">
                            <Button label="Adicionar" icon="pi pi-plus" class="p-button-success mr-2" @click="openNew" />
                            <Button label="Excluir" icon="pi pi-trash" class="p-button-danger" @click="confirmDeleteSelected" :disabled="!selectedProducts || !selectedProducts.length" />
                        </div>
                    </template>
                </Toolbar>

                <DataTable
                    ref="dt"
                    :value="products"
                    v-model:selection="selectedProducts"
                    dataKey="id"
                    :paginator="true"
                    :rows="10"
                    :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Mostrando {first} até {last} de {totalRecords} agências"
                    responsiveLayout="scroll"
                >
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Gerenciamento das agências</h5>
                            <span class="block mt-2 md:mt-0 p-input-icon-left">
                                <i class="pi pi-search" />
                                <InputText v-model="filters['global'].value" placeholder="Procurar..." />
                            </span>
                        </div>
                    </template>

                    <Column selectionMode="multiple" headerStyle="width: 3rem"></Column>
                    <Column field="razaoSocial" header="Razão Social" :sortable="true" headerStyle="width:25%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Razão Social</span>
                            {{ slotProps.data.razaoSocial }}
                        </template>
                    </Column>
                    <Column field="cnpj" header="CNPJ" :sortable="true" headerStyle="width:20%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">CNPJ</span>
                            {{ slotProps.data.cnpj }}
                        </template>
                    </Column>
                    <Column field="telefone" header="Telefone" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Telefone</span>
                            {{ slotProps.data.telefone }}
                        </template>
                    </Column>
                    <Column field="email" header="Email" :sortable="true" headerStyle="width:25%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Email</span>
                            {{ slotProps.data.email }}
                        </template>
                    </Column>
                    <Column headerStyle="min-width:10rem;">
                        <template #body="slotProps">
                            <Button icon="pi pi-pencil" class="p-button-rounded p-button-success mr-2" @click="editProduct(slotProps.data)" />
                            <Button icon="pi pi-trash" class="p-button-rounded p-button-warning mt-2" @click="confirmDeleteProduct(slotProps.data)" />
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="productDialog" :style="{ width: '450px' }" header="Cadastro" :modal="true" class="p-fluid">
                    <div class="field">
                        <label for="razaoSocial">Razão Social</label>
                        <InputText id="razaoSocial" v-model.trim="product.razaoSocial" required="true" autofocus :class="{ 'p-invalid': submitted && !product.razaoSocial }" />
                        <small class="p-invalid" v-if="submitted && !product.razaoSocial">Razão Social é necessária.</small>
                    </div>
                    <div class="field">
                        <label for="cnpj">CNPJ</label>
                        <InputText id="cnpj" v-model.trim="product.cnpj" required="true" @mask="cnpjMask" @blur="validateCNPJ" />
                    </div>
                    <div class="field">
                        <label for="telefone">Telefone</label>
                        <InputText id="telefone" v-model.trim="product.telefone" required="true" @mask="phoneMask" @blur="validateTelefone" />
                    </div>
                    <div class="field">
                        <label for="email">Email</label>
                        <InputText id="email" v-model.trim="product.email" required="true" :class="{ 'p-invalid': submitted && !product.email }" />
                        <small class="p-invalid" v-if="submitted && !product.email">Email é necessário.</small>
                    </div>
                    <div class="field">
                        <label for="endereco"><b>Endereço:</b></label>
                        <InputTextarea id="endereco" v-model="product.endereco" rows="3" cols="20" :required="true" />
                    </div>
                    <div class="field">
                        <label for="cep">CEP</label>
                        <InputText id="cep" v-model.trim="product.cep" @blur="fetchAddressByPostalCode(product.cep)" required="true" :class="{ 'p-invalid': submitted && !product.cep }" />
                        <small class="p-invalid" v-if="submitted && !product.cep">CEP é necessário.</small>
                    </div>
                    <div class="field">
                        <label for="rua">Rua</label>
                        <InputText id="rua" v-model.trim="product.rua" required="true" :class="{ 'p-invalid': submitted && !product.rua }" />
                        <small class="p-invalid" v-if="submitted && !product.rua">Rua é necessário.</small>
                    </div>
                    <div class="field">
                        <label for="numero">Número</label>
                        <InputText id="numero" v-model.trim="product.numero" required="true" :class="{ 'p-invalid': submitted && !product.numero }" />
                        <small class="p-invalid" v-if="submitted && !product.numero">Número é necessário.</small>
                    </div>
                    <div class="field">
                        <label for="bairro">Bairro</label>
                        <InputText id="bairro" v-model.trim="product.bairro" required="true" :class="{ 'p-invalid': submitted && !product.bairro }" />
                        <small class="p-invalid" v-if="submitted && !product.bairro">Bairro é necessário.</small>
                    </div>
                    <div class="field">
                        <label for="cidade">Cidade</label>
                        <InputText id="cidade" v-model.trim="product.cidade" required="true" :class="{ 'p-invalid': submitted && !product.cidade }" />
                        <small class="p-invalid" v-if="submitted && !product.cidade">Cidade é necessário.</small>
                    </div>
                    <div class="field">
                        <label for="complemento">Complemento</label>
                        <InputText id="complemento" v-model.trim="product.complemento"/>
                    </div>

                    <template #footer>
                        <Button label="Cancelar" icon="pi pi-times" class="p-button-text" @click="hideDialog" />
                        <Button label="Salvar" icon="pi pi-check" class="p-button-text" @click="saveProduct" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="deleteProductDialog" :style="{ width: '450px' }" header="Confirmar" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="product"
                            >Você deseja excluir <b>{{ product.razaoSocial }}</b
                            >?</span
                        >
                    </div>
                    <template #footer>
                        <Button label="Não" icon="pi pi-times" class="p-button-text" @click="deleteProductDialog = false" />
                        <Button label="Sim" icon="pi pi-check" class="p-button-text" @click="deleteProduct" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="deleteProductsDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="product">Are you sure you want to delete the selected companies?</span>
                    </div>
                    <template #footer>
                        <Button label="No" icon="pi pi-times" class="p-button-text" @click="deleteProductsDialog = false" />
                        <Button label="Yes" icon="pi pi-check" class="p-button-text" @click="deleteSelectedProducts" />
                    </template>
                </Dialog>
            </div>
        </div>
    </div>
</template>
