<template>
	<div>
		<h3 class="text-lg-bold text-heading">
			{{ t('dashboard.summary.editorsAssigned') }}:
		</h3>
		<ul
			v-if="summaryStore.associatedEditors?.length"
			class="mt-2 flex flex-col space-y-2"
		>
			<li
				v-for="editor in summaryStore.associatedEditors"
				:key="editor.id"
				class="text-base-normal"
			>
				<div class="flex">
					<div>
						<UserAvatar
							:user-id="editor.id"
							:user-full-name="editor.fullName"
						></UserAvatar>
					</div>
					<div class="ms-2 flex flex-col justify-center">
						<div class="text-base-bold">{{ editor.fullName }}</div>
						<div class="text-sm-normal text-secondary">
							{{ editor.roleName }}
						</div>
					</div>
				</div>
			</li>
		</ul>
		<span v-else class="-ms-3">
			<PkpButton
				is-link="true"
				@click="() => summaryStore.handleAction('assignParticipant')"
			>
				Assign Editors
			</PkpButton>
		</span>
	</div>
</template>
<script setup>
import UserAvatar from '@/components/UserAvatar/UserAvatar.vue';
import PkpButton from '@/components/Button/Button.vue';
import {useLocalize} from '@/composables/useLocalize';
import {useSubmissionSummaryStore} from '../submissionSummaryStore';

const {t} = useLocalize();
const summaryStore = useSubmissionSummaryStore();
</script>
