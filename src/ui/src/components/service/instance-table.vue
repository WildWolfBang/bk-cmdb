<template>
    <div class="service-table-layout">
        <div class="title" @click="localExpanded = !localExpanded">
            <div class="fl">
                <i class="bk-icon icon-down-shape" v-if="localExpanded"></i>
                <i class="bk-icon icon-right-shape" v-else></i>
                {{name}}
            </div>
            <div class="fr right-content">
                <span v-if="topology" class="service-topology" :title="topology">{{topology}}</span>
                <i class="bk-icon icon-close" v-if="deletable" @click.stop="handleDelete"></i>
            </div>
        </div>
        <bk-table class="service-table"
            v-show="localExpanded"
            :data="processList">
            <bk-table-column v-for="column in header"
                :key="column.id"
                :prop="column.id"
                :label="column.name">
                <template slot-scope="{ row }">{{row[column.id] | formatter(column.property)}}</template>
            </bk-table-column>
            <bk-table-column :label="$t('操作')" fixed="right">
                <template slot-scope="{ row, $index }">
                    <a href="javascript:void(0)" class="text-primary mr10" @click="handleEditProcess($index)">
                        {{$t('编辑')}}
                    </a>
                    <a href="javascript:void(0)" class="text-primary"
                        v-if="!sourceProcesses.length"
                        @click="handleDeleteProcess($index)">
                        {{$t('删除')}}
                    </a>
                </template>
            </bk-table-column>
            <template slot="empty" v-if="addible">
                <button class="add-process-button text-primary" @click="handleAddProcess">
                    <i class="bk-icon icon-plus"></i>
                    <span>{{$t('添加进程')}}</span>
                </button>
            </template>
        </bk-table>
        <div class="add-process-options" v-if="localExpanded && addible && !sourceProcesses.length && processList.length">
            <button class="add-process-button text-primary" @click="handleAddProcess">
                <i class="bk-icon icon-plus"></i>
                <span>{{$t('添加进程')}}</span>
            </button>
        </div>
    </div>
</template>

<script>
    import { processTableHeader } from '@/dictionary/table-header'
    import {
        processPropertyRequestId,
        processPropertyGroupsRequestId
    } from './form/symbol'
    import Form from './form/form.js'
    export default {
        props: {
            deletable: Boolean,
            expanded: Boolean,
            id: {
                type: Number,
                required: true
            },
            index: {
                type: Number,
                required: true
            },
            name: {
                type: String,
                default: ''
            },
            sourceProcesses: {
                type: Array,
                default () {
                    return []
                }
            },
            templates: {
                type: Array,
                default () {
                    return []
                }
            },
            addible: {
                type: Boolean,
                default: true
            },
            topology: {
                type: String,
                default: ''
            }
        },
        data () {
            return {
                localExpanded: this.expanded,
                processList: this.$tools.clone(this.sourceProcesses),
                processProperties: [],
                processPropertyGroups: [],
                tooltips: {
                    content: this.$t('请为主机添加进程'),
                    placement: 'right'
                }
            }
        },
        computed: {
            header () {
                const header = []
                processTableHeader.forEach(id => {
                    const property = this.processProperties.find(property => property.bk_property_id === id)
                    if (property) {
                        header.push({
                            id: property.bk_property_id,
                            name: this.$tools.getHeaderPropertyName(property),
                            property
                        })
                    }
                })
                return header
            }
        },
        created () {
            this.getProcessProperties()
            this.getProcessPropertyGroups()
        },
        methods: {
            async getProcessProperties () {
                try {
                    const action = 'objectModelProperty/searchObjectAttribute'
                    this.processProperties = await this.$store.dispatch(action, {
                        params: {
                            bk_obj_id: 'process',
                            bk_supplier_account: this.$store.getters.supplierAccount
                        },
                        config: {
                            requestId: processPropertyRequestId,
                            fromCache: true
                        }
                    })
                } catch (e) {
                    console.error(e)
                }
            },
            async getProcessPropertyGroups () {
                try {
                    const action = 'objectModelFieldGroup/searchGroup'
                    this.processPropertyGroups = await this.$store.dispatch(action, {
                        objId: 'process',
                        params: {},
                        config: {
                            requestId: processPropertyGroupsRequestId,
                            fromCache: true
                        }
                    })
                } catch (e) {
                    console.error(e)
                }
            },
            handleDelete () {
                this.$emit('delete-instance', this.index)
            },
            handleAddProcess () {
                Form.show({
                    type: 'create',
                    title: this.$t('添加进程'),
                    hostId: this.id,
                    submitHandler: values => {
                        this.processList.push(values)
                    }
                })
            },
            handleEditProcess (rowIndex) {
                Form.show({
                    type: 'update',
                    title: this.$t('编辑进程'),
                    instance: this.processList[rowIndex],
                    serviceTemplateId: this.templates[rowIndex] ? this.templates[rowIndex].service_template_id : 0,
                    processTemplateId: this.templates[rowIndex] ? this.templates[rowIndex].id : 0,
                    hostId: this.id,
                    submitHandler: (values, changedValues, raw) => {
                        Object.assign(raw, changedValues)
                    }
                })
            },
            handleDeleteProcess (rowIndex) {
                this.processList.splice(rowIndex, 1)
            }
        }
    }
</script>

<style lang="scss" scoped>
    .title {
        height: 40px;
        padding: 0 10px;
        line-height: 40px;
        border-radius: 2px 2px 0 0;
        background-color: #DCDEE5;
        cursor: pointer;
        .bk-icon {
            font-size: 12px;
            font-weight: bold;
            width: 24px;
            height: 24px;
            line-height: 24px;
            text-align: center;
            cursor: pointer;
            @include inlineBlock;
            &.icon-close {
                font-size: 20px;
            }
        }
        .icon-exclamation {
            font-size: 14px;
            color: #ffffff;
            background: #f0b659;
            border-radius: 50%;
            transform: scale(.6);
        }
        .right-content {
            max-width: 70%;
            @include ellipsis;
        }
        .service-topology {
            padding: 0 5px;
            line-height: 40px;
            font-size: 12px;
            color: $textColor;
            cursor: default;
        }
    }
    .add-process-options {
        border: 1px solid $cmdbTableBorderColor;
        border-top: none;
        line-height: 42px;
        font-size: 12px;
        text-align: center;
    }
    .add-process-button {
        line-height: 32px;
        .bk-icon,
        span {
            @include inlineBlock;
        }
        .icon-plus {
            font-size: 20px;
            margin-right: -4px;
        }
    }
    .service-table {
        /deep/ {
            .bk-table-empty-block {
                min-height: 42px;
                .bk-table-empty-text {
                    padding: 0;
                }
            }
        }
    }
</style>
