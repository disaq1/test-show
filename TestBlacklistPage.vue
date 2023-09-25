<template>
    <BaseContainer
        :maxWidth="1040"
        class="mt-16"
    >
        <h1 class="display-small | neutral10--text mb-9">
            Чёрный список
        </h1>
        <BlacklistSkeleton v-if="isBlacklistLoading" />
        <template v-else>
            <ErrorsComponent
                v-if="blacklistErrorMessage"
                :text="blacklistErrorMessage"
            />
            <ErrorsComponent
                v-else-if="!blacklist.length"
                :icon="'mdi-message-lock-outline'"
                :title="'Черный список пуст'"
                :text="'Вы можете запретить отправку сообщений абонентам добавив их номер телефона в черный список'"
            >
                <template #button>
                    <BaseButton
                        class="mt-10"
                        @click.prevent="isOpenedImportModal = true"
                    >
                        Добавить номер
                    </BaseButton>
                </template>
            </ErrorsComponent>
            <v-row v-else>
                <v-col
                    cols="12"
                    md="4"
                >
                    <BlacklistCsv
                        :is-table-loading="isTableLoading"
                        :filters="filters"
                        @delete-selected="openDeleteModal($event)"
                    />
                </v-col>
                <v-col
                    cols="12"
                    md="8"
                >
                    <BlacklistTable
                        :blacklist="filteredBlacklist"
                        :is-table-loading="isTableLoading"
                        :filters="filters"
                        :blacklist-length="blacklistLength"
                        @delete-selected="openDeleteModal($event)"
                        @filter-applied="applyFilter($event)"
                        @open-import-modal="isOpenedImportModal = true"
                    />
                </v-col>
            </v-row>
        </template>
        <BlacklistImportModal
            v-if="isOpenedImportModal"
            v-model="isOpenedImportModal"
            :is-table-loading="isTableLoading"
            @file-imported="importFile($event)"
            @modal-closed="isOpenedImportModal = false"
            @phone-numbers-imported="importPhoneNumbers($event)"
        />
        <BlacklistResultModal
            v-if="isOpenedResultModal"
            v-model="isOpenedResultModal"
            :error-message="importFileError"
            :import-file-result="importFileResult"
            :is-table-loading="isTableLoading"
            @modal-closed="isOpenedResultModal = false"
            @repeat-clicked="reopenImportModal()"
        />
        <BlacklistDeleteModal
            v-if="isOpenedDeleteModal"
            v-model="isOpenedDeleteModal"
            :is-loading="isTableLoading"
            @delete-confirmed="deleteSelected(selected)"
            @modal-closed="isOpenedDeleteModal = false"
        >
            <template #title>
                {{ deleteModal.title }}
            </template>
            <template #text>
                {{ deleteModal.text }}
            </template>
            <template #buttonTextCancel>
                {{ deleteModal.buttonTextCancel }}
            </template>
            <template #buttonTextConfirm>
                {{ deleteModal.buttonTextConfirm }}
            </template>
        </BlacklistDeleteModal>
    </BaseContainer>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useSnackbar } from '@/composables/snackbar';
const { showSnackbar } = useSnackbar();
import axios from '@/mixins/axios-instance';
import BlacklistCsv from '@/views/cabinet/blacklist/BlacklistCsv';
import BlacklistDeleteModal from '@/views/cabinet/blacklist/BlacklistDeleteModal';
import BlacklistImportModal from '@/views/cabinet/blacklist/BlacklistImportModal';
import BlacklistResultModal from '@/views/cabinet/blacklist/BlacklistResultModal';
import BlacklistSkeleton from '@/views/cabinet/blacklist/BlacklistSkeleton';
import BlacklistTable from '@/views/cabinet/blacklist/BlacklistTable';
import ErrorsComponent from '@/components/ErrorsComponent';

const defaultFilters = () => ({
    page: 1,
    per_page: 10,
    search: '',
    sorter: 'asc',
});
const defaultImportFileResult = () => ({
    addedPhones: 0,
    allPhones: 0,
    badPhones: [],
    existPhones: [],
});

const blacklist = ref([]);
const blacklistLength = ref(0);
const blacklistErrorMessage = ref('');
const defaultBlacklistError = 'Произошла ошибка при загрузке страницы. Пожалуйста, попробуйте позже';
const defaultDeleteError = 'Произошла ошибка при удалении номеров из чёрного списка. Пожалуйста, попробуйте позже';
const defaultImportError = 'Произошла ошибка при импорте. Пожалуйста, попробуйте позже';
const deleteModal = ref({
    buttonTextCancel: '',
    buttonTextConfirm: '',
    text: '',
    title: '',
});
const filteredBlacklist = ref([]);
const filters = ref({
    page: 1,
    per_page: 10,
    search: '',
    sorter: 'asc',
});
const importFileError = ref('');
const importFileResult = ref(defaultImportFileResult());
const isBlacklistLoading = ref(false);
const isOpenedDeleteModal = ref(false);
const isOpenedImportModal = ref(false);
const isOpenedResultModal = ref(false);
const isTableLoading = ref(false);
const selected = ref([]);

onMounted(async () => getBlacklist());

const setDeleteModalText = (selected) => {
    if (selected.length) {
        deleteModal.value.title = 'Удалить номера?';
        deleteModal.value.text = `Подтвердите удаление ${selected.length } ${selected.length === 1 ? 'выбранного номера' : 'выбранных номеров'}`;
        deleteModal.value.buttonTextCancel = 'Отменить';
        deleteModal.value.buttonTextConfirm = 'Удалить';
    } else {
        deleteModal.value.title = 'Очистить черный список?';
        deleteModal.value.text = 'Подтвердите удаление всех номеров телефонов из черного списка';
        deleteModal.value.buttonTextCancel = 'Закрыть';
        deleteModal.value.buttonTextConfirm = 'Очистить';
    }
};
const openDeleteModal = (selectedArray) => {
    selected.value = [];
    selected.value = selectedArray;
    setDeleteModalText(selectedArray);
    isOpenedDeleteModal.value = true;
};
const reopenImportModal = () => {
    importFileResult.value = defaultImportFileResult();
    isOpenedResultModal.value = false;
    isOpenedImportModal.value = true;
};
const importFile = async (file) => {
    isOpenedImportModal.value = false;
    isTableLoading.value = true;
    isOpenedResultModal.value = true;
    importFileError.value = '';

    try {
        let formData = new FormData();
        formData.append('blacklist_file', file);

        const response = await axios.post('/api/cabinet/blacklist/addFromFile/', formData)

        if (response.data.status === 'ok') {
            importFileResult.value = response.data;

            blacklistLength.value ? await applyFilter(defaultFilters()) : await getBlacklist();
        } else {
            importFileError.value = response.data.errmsg ? response.data.errmsg : defaultImportError;
        }
    } catch (error) {
        console.error(error);
        showSnackbar(defaultImportError);
    } finally {
        isTableLoading.value = false;
    }
};
const importPhoneNumbers = async (content) => {
    isOpenedImportModal.value = false;
    isTableLoading.value = true;
    isOpenedResultModal.value = true;
    importFileError.value = '';

    try {
        const response = await axios.post('/api/cabinet/blacklist/add/', { content });

        if (response.data.status === 'ok') {
            importFileResult.value = response.data;

            blacklistLength.value ? await applyFilter(defaultFilters()) : await getBlacklist();
        } else {
            importFileError.value = response.data.errmsg ? response.data.errmsg : defaultImportError;
        }
    } catch (error) {
        console.error(error);
        showSnackbar(defaultImportError);
    } finally {
        isTableLoading.value = false;
    }
};
const getBlacklist = async () => {
    isBlacklistLoading.value = true;

    try {
        const response = await axios.post('/api/cabinet/blacklist/stats', JSON.stringify(defaultFilters()), {
            headers: {
                'Content-Type': 'application/json',
            },
        });

        if (response.data.status === 'ok') {
            filters.value = defaultFilters();
            blacklist.value = response.data.list;
            filteredBlacklist.value = response.data.list;
            blacklistLength.value = Number(response.data.stats.all);
        } else {
            blacklistErrorMessage.value = response.data.errmsg ? response.data.errmsg : defaultBlacklistError;
        }
    } catch (error) {
        console.error(error);
        blacklistErrorMessage.value = error.message ? error.message : defaultBlacklistError;
    } finally {
        isBlacklistLoading.value = false;
    }
};
const deleteSelected = async (selected) => {
    if (!selected.length) {
        await deleteSelectedAll();
        return;
    }

    const phonesList = selected.map(item => item.BL_PHONE);

    isTableLoading.value = true;

    try {
        const response = await axios.post(
            '/api/cabinet/blacklist/delete',
            {
                phones: phonesList,
            },
            {
                headers: {
                    'Content-Type': 'application/json',
                },
            },
        );

        if (response.data.status === 'ok') {
            showSnackbar(`${phonesList.length > 1 ? 'Номера успешно удалены' : 'Номер успешно удалён'} из чёрного списка`);

            await getBlacklist();
        } else {
            showSnackbar(response.data.errmsg ? response.data.errmsg : defaultDeleteError);
        }
    } catch (error) {
        console.error(error);
        showSnackbar(defaultDeleteError);

    } finally {
        isTableLoading.value = false;
        isOpenedDeleteModal.value = false;
    }
};
const deleteSelectedAll = async () => {
    isTableLoading.value = true;

    try {
        const response = await axios.post('/api/cabinet/blacklist/deleteAll');

        if (response.data.status === 'ok') {
            showSnackbar('Номера удалены из чёрного списка');

            await getBlacklist();
        } else {
            showSnackbar(response.data.errmsg ? response.data.errmsg : defaultDeleteError);
        }
    } catch (error) {
        console.error(error);
        showSnackbar(defaultDeleteError);
    } finally {
        isTableLoading.value = false;
        isOpenedDeleteModal.value = false;
    }
};
const applyFilter = async (filter = defaultFilters()) => {
    isTableLoading.value = true;
    filteredBlacklist.value = [];

    try {
        const response = await axios.post('/api/cabinet/blacklist/stats', JSON.stringify(filter), {
            headers: {
                'Content-Type': 'application/json',
            },
        });

        if (response.data.status === 'ok') {
            filters.value = filter;
            filteredBlacklist.value = response.data.list;
            blacklistLength.value = Number(response.data.stats.all);
        } else {
            blacklistErrorMessage.value = response.data.errmsg ? response.data.errmsg : defaultBlacklistError;
        }
    } catch (error) {
        console.error(error);
        blacklistErrorMessage.value = error.message ? error.message : defaultBlacklistError;
    } finally {
        isTableLoading.value = false;
    }
};
</script>
