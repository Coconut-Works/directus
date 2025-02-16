<template>
	<div>
		<div class="grid">
			<div class="field">
				<div class="type-label">{{ t('this_collection') }}</div>
				<v-input disabled :model-value="collection" />
			</div>

			<div class="field">
				<div class="type-label">{{ t('related_collection') }}</div>
				<related-collection-select v-model="relatedCollection" :disabled="isExisting" />
			</div>

			<v-input disabled :model-value="currentPrimaryKey" />

			<related-field-select v-model="relatedField" :collection="relatedCollection" />

			<v-icon class="arrow" name="arrow_forward" />
		</div>

		<div class="sort-field">
			<v-divider large :inline-title="false">{{ t('sort_field') }}</v-divider>

			<related-field-select
				v-model="sortField"
				:collection="relatedCollection"
				:placeholder="t('add_sort_field') + '...'"
			/>
		</div>

		<div class="relational-triggers">
			<v-divider class="field full" large :inline-title="false">{{ t('relational_triggers') }}</v-divider>

			<div class="field">
				<div class="type-label">
					{{
						t('referential_action_field_label_o2m', {
							collection: relatedCollection || 'related',
						})
					}}
				</div>
				<v-select
					v-model="onDeselect"
					:placeholder="t('choose_action') + '...'"
					:items="[
						{
							text: t('referential_action_set_null', { field: relatedField }),
							value: 'nullify',
						},
						{
							text: t('referential_action_cascade', {
								collection: relatedCollection,
								field: relatedField,
							}),
							value: 'delete',
						},
					]"
				/>
			</div>

			<div class="field">
				<div class="type-label">
					{{
						t('referential_action_field_label_m2o', {
							collection: collection || 'related',
						})
					}}
				</div>
				<v-select
					v-model="onDelete"
					:disabled="collection === relatedCollection"
					:placeholder="t('choose_action') + '...'"
					:items="[
						{
							text: t('referential_action_set_null', { field: relatedField }),
							value: 'SET NULL',
						},
						{
							text: t('referential_action_set_default', { field: relatedField }),
							value: 'SET DEFAULT',
						},
						{
							text: t('referential_action_cascade', {
								collection: collection,
								field: relatedField,
							}),
							value: 'CASCADE',
						},
						{
							text: t('referential_action_no_action'),
							value: 'NO ACTION',
						},
					]"
				/>
			</div>
		</div>

		<v-notice v-if="generationInfo.length > 0" class="generated-data" type="warning">
			<span>
				{{ t('new_data_alert') }}

				<ul>
					<li v-for="(data, index) in generationInfo" :key="index">
						<span class="field-name">{{ data.name }}</span>
					</li>
				</ul>
			</span>
		</v-notice>
	</div>
</template>

<script lang="ts">
import { useI18n } from 'vue-i18n';
import { defineComponent, computed } from 'vue';
import { useFieldDetailStore, syncFieldDetailStoreProperty } from '../store';
import { storeToRefs } from 'pinia';
import RelatedCollectionSelect from '../shared/related-collection-select.vue';
import RelatedFieldSelect from '../shared/related-field-select.vue';
import { useFieldsStore } from '@/stores';

export default defineComponent({
	components: { RelatedCollectionSelect, RelatedFieldSelect },
	setup() {
		const { t } = useI18n();

		const fieldDetailStore = useFieldDetailStore();
		const fieldsStore = useFieldsStore();

		const relatedCollection = syncFieldDetailStoreProperty('relations.o2m.collection');
		const relatedField = syncFieldDetailStoreProperty('relations.o2m.field');
		const sortField = syncFieldDetailStoreProperty('relations.o2m.meta.sort_field');
		const onDelete = syncFieldDetailStoreProperty('relations.o2m.schema.on_delete');
		const onDeselect = syncFieldDetailStoreProperty('relations.o2m.meta.one_deselect_action');

		const { collection, editing, generationInfo } = storeToRefs(fieldDetailStore);

		const isExisting = computed(() => editing.value !== '+');
		const currentPrimaryKey = computed(() => fieldsStore.getPrimaryKeyFieldForCollection(collection.value!)?.field);

		return {
			t,
			isExisting,
			collection,
			relatedCollection,
			currentPrimaryKey,
			relatedField,
			generationInfo,
			sortField,
			onDelete,
			onDeselect,
		};
	},
});
</script>

<style lang="scss" scoped>
@import '@/styles/mixins/form-grid';

.grid {
	--v-select-font-family: var(--family-monospace);
	--v-input-font-family: var(--family-monospace);

	position: relative;
	display: grid;
	grid-template-columns: repeat(2, minmax(0, 1fr));
	gap: 12px 32px;
	margin-top: 48px;

	.v-icon.arrow {
		--v-icon-color: var(--primary);

		position: absolute;
		bottom: 17px;
		left: 50%;
		transform: translateX(-50%);
	}
}

.v-input.matches {
	--v-input-color: var(--primary);
}

.v-list {
	--v-list-item-content-font-family: var(--family-monospace);
}

.type-label {
	margin-bottom: 8px;
}

.v-divider {
	margin: 48px 0;
}

.corresponding {
	position: relative;
	display: grid;
	grid-template-columns: repeat(2, minmax(0, 1fr));
	gap: 12px 32px;
	margin-top: 48px;

	.arrow {
		--v-icon-color: var(--primary);

		position: absolute;
		bottom: 17px;
		left: 50%;
		transform: translateX(-50%);
	}
}

.v-notice {
	margin-bottom: 36px;
}

.generated-data {
	margin-top: 36px;

	ul {
		padding-top: 4px;
		padding-left: 24px;
	}

	.field-name {
		font-family: var(--family-monospace);
	}
}

.sort-field {
	--v-input-font-family: var(--family-monospace);

	.v-divider {
		margin-top: 48px;
		margin-bottom: 24px;
	}
}

.relational-triggers {
	--form-horizontal-gap: 12px;
	--form-vertical-gap: 24px;

	@include form-grid;

	.v-divider {
		margin-top: 48px;
		margin-bottom: 0;
	}
}
</style>
