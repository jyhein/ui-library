<template>
	<div>
		<SideModalBody>
			<template #title>
				{{ title }}
			</template>
			<SideModalLayoutBasic>
				<PkpForm
					ref="editRole"
					class="contributorRoles__contributorRoleForm"
					v-bind="formProps"
					@success="
						(...args) => {
							$emit('contributorRoleSaved', ...args);
							onRoleSaved();
						}
					"
				></PkpForm>
			</SideModalLayoutBasic>
		</SideModalBody>
	</div>
</template>

<script setup>
import SideModalBody from '@/components/Modal/SideModalBody.vue';
import SideModalLayoutBasic from '@/components/Modal/SideModalLayoutBasic.vue';
import PkpForm from '@/components/Form/Form.vue';

defineProps({
	title: {type: String, required: true},
	formProps: {ype: Object, required: true},
	onRoleSaved: {type: Function, required: true},
});

defineEmits(['contributorRoleSaved']);
</script>
