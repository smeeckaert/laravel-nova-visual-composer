<template>
    <default-field :field="field" :errors="errors" :full-width-content="true">
        <template slot="field">

            <draggable
                v-model="rows"
                :options="{ handle: '.js-row-item-move' }"
                @end="refreshValue"
            >
                <row
                    v-for="(row, index) in rows"
                    v-model="rows[index]"
                    :key="row.index"
                    :index="row.index"
                    :type="row.type"
                    :initialValue="row.initialValue"
                    @delete-row="deleteRow($event)"
                    @update-row="updateRow($event)"
                ></row>
            </draggable>

            <div class="actions">
                <select v-model="templateSelector" class="w-full form-control form-select mb-2">
                    <option
                        value=""
                        v-text=""
                        selected
                        disabled
                    >
                        Choisir un template
                    </option>
                    <option
                        v-for="template in field.templates"
                        :value="template.classname"
                        v-text="template.name_trans"
                    ></option>
                </select>

                <button
                    class="btn btn-default btn-primary"
                    @click.prevent="addNewRow(templateSelector)"
                    v-text="field.addRowButtonLabel"
                ></button>
            </div>
        </template>
    </default-field>
</template>

<script>
    import draggable from 'vuedraggable'
    import Row from './rows/Row.vue'
    import Vue from 'vue';
    import {FormField, HandlesValidationErrors} from 'laravel-nova'

    export default {
        mixins: [FormField, HandlesValidationErrors],

        components: {
            draggable,
            Row
        },

        props: ['resourceName', 'resourceId', 'field'],

        data: () => ({
            value: '',
            rows: [],
            counter: 0,
        }),

        computed: {
            addButtonText() {
                return (this.field.add_button_text)
                    ? this.field.add_button_text
                    : 'Add row'
            },

            templateLabel(template) {
                return (template.name !== template.name_trans) ? template.name_trans : template.name;
            }
        },

        methods: {
            /**
             * Set the initial, internal value for the field.
             */
            setInitialValue() {
                this.value = this.field.value || '';
                this.$nextTick(() => {
                    if (this.value !== '') {
                        this.resetRowsFromValue();
                    }
                });
            },
            /**
             * Reset rows and create them from value
             */
            resetRowsFromValue() {
                this.rows = [];

                const rows = JSON.parse(this.value);
                rows.forEach((row) => {
                    this.addNewRow(row.template, row.content);
                });
            },
            /**
             * Fill the given FormData object with the field's internal value.
             */
            fill(formData) {
                formData.append(this.field.attribute, this.value || '')
            },
            /**
             * Update the field's internal value.
             */
            handleChange(value) {
                this.value = value
            },
            /**
             * Create a new new
             * @param rowType
             * @param content
             */
            addNewRow(rowType, content) {
                if (!rowType) {
                    return;
                }

                const newRow = new Vue({
                    ...Row,
                    propsData: {
                        type: rowType,
                        initialValue: (content ? content : ''),
                        index: this.counter,
                    }
                });

                this.counter ++;

                this.rows.push(newRow);

                this.refreshValue();
            },

            deleteRow(event) {

                const index = this.rows.findIndex(function(row) {
                    return row.index === event[0];
                });

                if (index !== -1) {
                    // Delete selected row
                    this.$delete(this.rows, index);
                    // Refresh field value
                    this.refreshValue();
                }
            },

            updateRow(event) {

                const index = this.rows.findIndex(function(row) {
                    return row.index === event[0];
                });

                if (index !== -1) {
                    // Update value of current row updated
                    this.rows[index]['initialValue'] = event[1];
                    // Update field value
                    this.refreshValue();
                }
            },

            refreshValue() {
                let contents = [];
                this.rows.forEach((row) => {
                    contents.push({
                        template: row.type,
                        content: row.initialValue,
                    });
                });

                this.value = JSON.stringify(contents);
            },
        },
    }
</script>
