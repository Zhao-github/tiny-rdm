<script setup>
import { reactive, ref, watch } from 'vue'
import useDialog from 'stores/dialog'
import useTabStore from 'stores/tab.js'
import useConnectionStore from 'stores/connections.js'

const ttlForm = reactive({
    ttl: -1,
})

const dialogStore = useDialog()
const connectionStore = useConnectionStore()
const tabStore = useTabStore()

const currentServer = ref('')
const currentKey = ref('')
const currentDB = ref(0)
watch(
    () => dialogStore.ttlDialogVisible,
    (visible) => {
        if (visible) {
            // get ttl from current tab
            const tab = tabStore.currentTab
            if (tab != null) {
                ttlForm.key = tab.key
                if (tab.ttl < 0) {
                    // forever
                } else {
                    ttlForm.ttl = tab.ttl
                }
                currentServer.value = tab.name
                currentDB.value = tab.db
                currentKey.value = tab.key
            }
        }
    },
)

const onClose = () => {
    dialogStore.closeTTLDialog()
}

const onConfirm = async () => {
    try {
        const tab = tabStore.currentTab
        if (tab == null) {
            return
        }
        const success = await connectionStore.setTTL(tab.name, tab.db, tab.key, ttlForm.ttl)
        if (success) {
            tabStore.updateTTL({
                server: currentServer.value,
                db: currentDB.value,
                key: currentKey.value,
                ttl: ttlForm.ttl,
            })
        }
    } catch (e) {
    } finally {
        dialogStore.closeTTLDialog()
    }
}
</script>

<template>
    <n-modal
        v-model:show="dialogStore.ttlDialogVisible"
        :closable="false"
        :close-on-esc="false"
        :mask-closable="false"
        :show-icon="false"
        :title="$t('dialogue.ttl.title')"
        preset="dialog"
        transform-origin="center">
        <n-form :model="ttlForm" :show-require-mark="false" label-placement="top">
            <n-form-item :label="$t('common.key')">
                <n-input :value="currentKey" readonly />
            </n-form-item>
            <n-form-item :label="$t('interface.ttl')" required>
                <n-input-number
                    v-model:value="ttlForm.ttl"
                    :max="Number.MAX_SAFE_INTEGER"
                    :min="-1"
                    style="width: 100%">
                    <template #suffix>
                        {{ $t('common.second') }}
                    </template>
                </n-input-number>
            </n-form-item>
        </n-form>

        <template #action>
            <div class="flex-item-expand">
                <n-button :focusable="false" @click="ttlForm.ttl = -1">{{ $t('dialogue.key.persist_key') }}</n-button>
            </div>
            <div class="flex-item n-dialog__action">
                <n-button :focusable="false" @click="onClose">{{ $t('common.cancel') }}</n-button>
                <n-button type="primary" :focusable="false" @click="onConfirm">{{ $t('common.save') }}</n-button>
            </div>
        </template>
    </n-modal>
</template>

<style lang="scss" scoped></style>
