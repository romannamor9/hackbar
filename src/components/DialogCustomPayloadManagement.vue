<template>
  <VDialog v-model="shown" max-width="100%" :width="680" persistent>
    <VCard>
      <VCardTitle> Manage Custom Payload </VCardTitle>
      <VCardText>
        <VDataTable
          :headers="headers"
          :items="allItems"
          :group-by="groupBy"
          :search="search"
        >
          <template #top>
            <VToolbar :class="$style.toolbar">
              <VTextField
                v-model="search"
                v-disable-prevent-default.keydown
                label="Search"
                variant="underlined"
                single-line
              />
              <VBtn variant="text" class="ml-2" @click="add"> Add </VBtn>
            </VToolbar>
          </template>
          <template #[`item.actions`]="{ item }">
            <VBtn variant="plain" icon @click="edit(item.raw)">
              <VIcon :icon="mdiPencil" />
            </VBtn>
            <VBtn variant="plain" icon @click="remove(item.raw)">
              <VIcon :icon="mdiDelete" />
            </VBtn>
          </template>
        </VDataTable>
      </VCardText>
      <VCardActions>
        <VSpacer />
        <VBtn variant="text" @click="shown = false"> Close </VBtn>
      </VCardActions>
    </VCard>
  </VDialog>
  <DialogCustomPayloadEdit
    v-if="editDialog.show"
    v-model="editDialog"
    @save="updatePayload"
  />
</template>

<script lang="ts">
import { mdiDelete, mdiPencil } from '@mdi/js'
import { storeToRefs } from 'pinia'
import { PropType, computed, defineComponent, ref, toRefs } from 'vue'
import { useCustomPayloadStore } from '../stores'
import { CustomPayloadTopLevel } from '../utils/constants'
import DialogCustomPayloadEdit from './DialogCustomPayloadEdit.vue'

export default defineComponent({
  name: 'DialogCustomPayloadManagement',
  components: {
    DialogCustomPayloadEdit,
  },
  props: {
    modelValue: {
      type: Object as PropType<{ show: boolean }>,
      required: true,
    },
  },
  emits: ['update:modelValue'],
  setup(props, { emit }) {
    const { modelValue } = toRefs(props)
    const shown = computed<boolean>({
      set: value => {
        emit(
          'update:modelValue',
          Object.assign({}, modelValue.value, { show: value }),
        )
      },
      get: () => {
        return modelValue.value.show
      },
    })

    const customPayloadStore = useCustomPayloadStore()
    const { allItems } = storeToRefs(customPayloadStore)
    const { save: savePayload, remove: removePayload } = customPayloadStore

    const editDialog = ref({
      show: false,
      payload: {
        name: '',
        value: '',
        category: CustomPayloadTopLevel,
      },
    })
    const updatePayload = async (payload: CustomPayload) => {
      await savePayload(
        allItems.value.indexOf(editDialog.value.payload),
        payload,
      )
    }

    const search = ref('')
    const headers = [
      {
        title: 'Name',
        key: 'name',
      },
      {
        title: 'Value',
        key: 'value',
      },
      {
        title: 'Actions',
        key: 'actions',
        align: 'center',
        sortable: false,
      },
    ] as const
    const groupBy = [{ key: 'category', order: false }]
    const add = () => {
      editDialog.value.show = true
      editDialog.value.payload = {
        name: '',
        value: '',
        category: CustomPayloadTopLevel,
      }
    }
    const edit = (payload: CustomPayload) => {
      editDialog.value.show = true
      editDialog.value.payload = payload
    }
    const remove = (payload: CustomPayload) => {
      removePayload(allItems.value.indexOf(payload))
    }

    return {
      mdiPencil,
      mdiDelete,

      shown,

      editDialog,
      updatePayload,

      search,
      headers,
      groupBy,
      allItems,
      add,
      edit,
      remove,
    }
  },
})
</script>

<style lang="scss" module>
.toolbar {
  border-radius: 5px !important;
  padding: 5px 10px;

  :global .v-toolbar__content > .v-btn:last-child {
    margin-inline-end: unset;
  }
}
</style>
