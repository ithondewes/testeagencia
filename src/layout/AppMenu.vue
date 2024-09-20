<script setup>
import { ref } from 'vue';

import AppMenuItem from './AppMenuItem.vue';

const model = ref([
    {
        to: '/pages',
        items: [
            {
                label: 'Perfil',
                icon: 'pi pi-fw pi-user',
                items: [
                    {
                        label: 'Impressoras',
                        to: '/uikit/list'
                    },
                    {
                        label: 'Usuários',
                        to: '/auth/error'
                    },
                ]
            }
        ]
    },
    {
        to: '/pages',
        items: [
            {
                label: 'Administrativo',
                icon: 'pi pi-fw pi-building',
                items: [
                    {
                        label: 'Cadastro',
                        to: '/uikit/list'
                    },
                    {
                        label: 'Cadastro 3º',
                        to: '/auth/error'
                    },
                    {
                        label: 'Estoque',
                        to: '/auth/access'
                    },
                    {
                        label: 'Financeiro',
                        to: '/auth/access'
                    },
                    {
                        label: 'Relatório',
                        to: '/auth/access'
                    }
                ]
            }
        ]
    },
    {
        to: '/pages',
        items: [
            {
                label: 'Ordem de Serviço',
                icon: 'pi pi-fw pi-file',
                items: [
                    {
                        label: 'Aprovação',
                        to: '/uikit/list'
                    },
                    {
                        label: 'Histórico',
                        to: '/auth/error'
                    },
                    {
                        label: 'Produção',
                        to: '/auth/access'
                    },
                    {
                        label: 'PCP',
                        to: '/auth/access'
                    },
                    {
                        label: 'Sistema Antigo',
                        to: '/auth/access'
                    }
                ]
            }
        ]
    },
    {
        to: '/pages',
        items: [
            {
                label: 'Gravação',
                icon: 'pi pi-fw pi-pencil',
                items: [
                    {
                        label: 'Estoque',
                        to: '/uikit/list'
                    },
                    {
                        label: 'Medidas',
                        to: '/auth/error'
                    }
                ]
            }
        ]
    },
    {
        items: [
            {
                label: 'Comercial',
                icon: 'pi pi-fw pi-briefcase',
                to: '/icons'
            }
        ]
    },
    {
        to: '/pages',
        items: [
            {
                label: 'Configurações',
                icon: 'pi pi-fw pi-cog',
                items: [
                    {
                        label: 'Permissões',
                        to: '/uikit/list'
                    },
                    {
                        label: 'Transportes',
                        to: '/auth/error'
                    }
                ]
            }
        ]
    }
]);
</script>

<template>
    <ul class="layout-menu">
        <template v-for="(item, i) in model" :key="item">
            <app-menu-item v-if="!item.separator" :item="item" :index="i"></app-menu-item>
            <li v-if="item.separator" class="menu-separator"></li>
        </template>
    </ul>
</template>

<style lang="scss" scoped></style>
