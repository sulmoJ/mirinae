<template>
    <p-dynamic-layout-table class="p-dynamic-layout-raw-table"
                            :name="name"
                            :options="{
                                fields,
                                translation_id: options.translation_id,
                                disable_search: options.disable_search,
                            }"
                            :data="rootData"
                            :fetch-options="fetchOptions"
                            :type-options="typeOptions"
                            :field-handler="fieldHandler"
                            v-on="$listeners"
    >
        <template v-for="(_, slot) of $scopedSlots"
                  #[slot]="scope"
        >
            <slot :name="slot"
                  v-bind="scope"
            />
        </template>
    </p-dynamic-layout-table>
</template>

<script lang="ts">
import type { PropType } from 'vue';
import {
    computed, defineComponent, reactive, toRefs,
} from 'vue';

import { map, sortBy } from 'lodash';

import type { DynamicFieldHandler } from '@/data-display/dynamic/dynamic-field/type';
import type { RawTableDynamicLayoutProps } from '@/data-display/dynamic/dynamic-layout/templates/raw-table/type';
import PDynamicLayoutTable from '@/data-display/dynamic/dynamic-layout/templates/table/index.vue';
import type { DynamicLayoutFetchOptions, DynamicLayoutTypeOptions } from '@/data-display/dynamic/dynamic-layout/type';
import type { RawTableOptions } from '@/data-display/dynamic/dynamic-layout/type/layout-schema';
import { getValueByPath } from '@/data-display/dynamic/helper';


export default defineComponent<RawTableDynamicLayoutProps>({
    name: 'PDynamicLayoutRawTable',
    components: {
        PDynamicLayoutTable,
    },
    props: {
        name: {
            type: String,
            default: '',
        },
        options: {
            type: Object as PropType<RawTableOptions>,
            default: () => ({}),
        },
        data: {
            type: [Object, Array, String],
            default: undefined,
        },
        fetchOptions: {
            type: Object as PropType<DynamicLayoutFetchOptions|undefined>,
            default: undefined,
        },
        typeOptions: {
            type: Object as PropType<DynamicLayoutTypeOptions|undefined>,
            default: undefined,
        },
        fieldHandler: {
            type: Function as PropType<DynamicFieldHandler|undefined>,
            default: undefined,
        },
    },
    setup(props) {
        const state = reactive({
            fields: computed(() => {
                if (state.rootData[0]) {
                    const firstItem = state.rootData[0];
                    if (Array.isArray(props.options?.headers) && props.options?.headers?.length) {
                        return props.options.headers.map((header) => ({
                            key: header,
                            name: header,
                        }));
                    }
                    return sortBy(map(firstItem, (value, key) => ({ key, name: key })), (item) => item.key);
                }
                return [];
            }),
            rootData: computed<any[]>(() => {
                if (props.options.root_path) {
                    const rootData = getValueByPath(props.data, props.options.root_path) ?? [];
                    return Array.isArray(rootData) ? rootData : [rootData];
                }
                if (Array.isArray(props.data)) return props.data;
                return [];
            }),
        });

        return {
            ...toRefs(state),
        };
    },
});
</script>
