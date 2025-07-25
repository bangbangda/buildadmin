<template>
    <div v-memo="[field]">
        <template v-for="(btn, idx) in field.buttons" :key="idx">
            <template v-if="btn.display ? btn.display(row, field) : true">
                <!-- 常规按钮 -->
                <el-button
                    v-if="btn.render == 'basicButton'"
                    v-blur
                    @click="onButtonClick(btn)"
                    :class="btn.class"
                    size="small"
                    class="ba-table-render-buttons-item"
                    :type="btn.type"
                    :loading="btn.loading && btn.loading(row, field)"
                    :disabled="btn.disabled && btn.disabled(row, field)"
                    v-bind="invokeTableContextDataFun(btn.attr, { row, field, cellValue: btn, column, index })"
                >
                    <Icon v-if="btn.icon" size="14" color="var(--ba-bg-color-overlay)" :name="btn.icon" />
                    <div v-if="btn.text" class="text">{{ getTranslation(btn.text) }}</div>
                </el-button>

                <!-- 带提示信息的按钮 -->
                <el-tooltip
                    v-if="btn.render == 'tipButton' && ((btn.name == 'edit' && baTable.auth('edit')) || btn.name != 'edit')"
                    :disabled="btn.title && !btn.disabledTip ? false : true"
                    :content="getTranslation(btn.title)"
                    placement="top"
                    v-bind="invokeTableContextDataFun(field.customRenderAttr?.tooltip, { row, field, cellValue: btn, column, index })"
                >
                    <el-button
                        v-blur
                        @click="onButtonClick(btn)"
                        :class="btn.class"
                        size="small"
                        class="ba-table-render-buttons-item"
                        :type="btn.type"
                        :loading="btn.loading && btn.loading(row, field)"
                        :disabled="btn.disabled && btn.disabled(row, field)"
                        v-bind="invokeTableContextDataFun(btn.attr, { row, field, cellValue: btn, column, index })"
                    >
                        <Icon v-if="btn.icon" size="14" color="var(--ba-bg-color-overlay)" :name="btn.icon" />
                        <div v-if="btn.text" class="text">{{ getTranslation(btn.text) }}</div>
                    </el-button>
                </el-tooltip>

                <!-- 带确认框的按钮 -->
                <el-popconfirm
                    v-if="btn.render == 'confirmButton' && ((btn.name == 'delete' && baTable.auth('del')) || btn.name != 'delete')"
                    :disabled="btn.disabled && btn.disabled(row, field)"
                    v-bind="invokeTableContextDataFun(btn.popconfirm, { row, field, cellValue: btn, column, index })"
                    @confirm="onButtonClick(btn)"
                >
                    <template #reference>
                        <div class="ml-6">
                            <el-tooltip
                                :disabled="btn.title ? false : true"
                                :content="getTranslation(btn.title)"
                                placement="top"
                                v-bind="invokeTableContextDataFun(field.customRenderAttr?.tooltip, { row, field, cellValue: btn, column, index })"
                            >
                                <el-button
                                    v-blur
                                    :class="btn.class"
                                    size="small"
                                    class="ba-table-render-buttons-item"
                                    :type="btn.type"
                                    :loading="btn.loading && btn.loading(row, field)"
                                    :disabled="btn.disabled && btn.disabled(row, field)"
                                    v-bind="invokeTableContextDataFun(btn.attr, { row, field, cellValue: btn, column, index })"
                                >
                                    <Icon v-if="btn.icon" size="14" color="var(--ba-bg-color-overlay)" :name="btn.icon" />
                                    <div v-if="btn.text" class="text">{{ getTranslation(btn.text) }}</div>
                                </el-button>
                            </el-tooltip>
                        </div>
                    </template>
                </el-popconfirm>

                <!-- 带提示的可拖拽按钮 -->
                <el-tooltip
                    v-if="btn.render == 'moveButton' && ((btn.name == 'weigh-sort' && baTable.auth('sortable')) || btn.name != 'weigh-sort')"
                    :disabled="btn.title && !btn.disabledTip ? false : true"
                    :content="getTranslation(btn.title)"
                    placement="top"
                    v-bind="invokeTableContextDataFun(field.customRenderAttr?.tooltip, { row, field, cellValue: btn, column, index })"
                >
                    <el-button
                        :class="btn.class"
                        size="small"
                        class="ba-table-render-buttons-item move-button"
                        :type="btn.type"
                        :loading="btn.loading && btn.loading(row, field)"
                        :disabled="btn.disabled && btn.disabled(row, field)"
                        v-bind="invokeTableContextDataFun(btn.attr, { row, field, cellValue: btn, column, index })"
                    >
                        <Icon v-if="btn.icon" size="14" color="var(--ba-bg-color-overlay)" :name="btn.icon" />
                        <div v-if="btn.text" class="text">{{ getTranslation(btn.text) }}</div>
                    </el-button>
                </el-tooltip>
            </template>
        </template>
    </div>
</template>

<script setup lang="ts">
import { TableColumnCtx } from 'element-plus'
import { inject } from 'vue'
import { useI18n } from 'vue-i18n'
import { invokeTableContextDataFun } from '/@/components/table/index'
import type baTableClass from '/@/utils/baTable'

interface Props {
    row: TableRow
    field: TableColumn
    column: TableColumnCtx<TableRow>
    index: number
}

const { t, te } = useI18n()
const props = defineProps<Props>()
const baTable = inject('baTable') as baTableClass

const onButtonClick = (btn: OptButton) => {
    if (typeof btn.click === 'function') {
        btn.click(props.row, props.field)
        return
    }
    baTable.onTableAction(btn.name, props)
}

const getTranslation = (key?: string) => {
    if (!key) return ''
    return te(key) ? t(key) : key
}
</script>

<style scoped lang="scss">
.ba-table-render-buttons-item {
    .text {
        padding-left: 5px;
    }
    &.el-button--small {
        padding: 4px 5px;
        height: auto;
    }
}
.ba-table-render-buttons-move {
    cursor: move;
}
.ml-6 {
    display: inline-flex;
    vertical-align: middle;
    margin-left: 6px;
}
.ml-6 + .el-button {
    margin-left: 6px;
}
</style>
