<template>
	<div
		v-if="!isDecidingEditorAssigned || (!isLoading && currentRecommendation)"
		class="w-full border border-light"
	>
		<div class="p-4">
			<h2 class="uppercase text-heading">
				{{ t('editor.submission.recommendation') }}
			</h2>
		</div>
		<div
			v-if="isDecidingEditorAssigned"
			class="flex flex-col border-t border-light p-4"
		>
			<p class="text-lg-normal text-default">
				{{ currentRecommendation.label }}
			</p>
			<span v-if="!showRecommendationActions" class="-ms-4 mt-2">
				<PkpButton class="" is-link @click="showActions">
					{{ t('editor.submission.workflowDecision.changeDecision') }}
				</PkpButton>
			</span>
		</div>
		<div v-else class="p-4">
			<p class="text-lg-normal text-default">
				{{ t('editor.submission.recommendation.noDecidingEditors') }}
			</p>
		</div>
	</div>
	<div v-if="showRecommendationActions" class="flex flex-col gap-y-2">
		<PkpButton
			v-for="actionItem in actionItems"
			:key="actionItem.action"
			:is-secondary="true"
			@click="() => handleAction(actionItem.action)"
		>
			{{ actionItem.label }}
		</PkpButton>
	</div>
</template>
<script setup>
import PkpButton from '@/components/Button/Button.vue';
import {computed, ref, watch} from 'vue';
import {useLocalize} from '@/composables/useLocalize';
import {useWorkflowStore} from '@/pages/workflow/workflowStore';
import {useUrl} from '@/composables/useUrl';
import {useFetch} from '@/composables/useFetch';

import {Actions as DecisionActions} from '../../composables/useWorkflowDecisions';
import {Actions as WorkflowActions} from '../../composables/useWorkflowActions';

const {t} = useLocalize();

const props = defineProps({
	submission: {type: Object, required: true},
	reviewRoundId: {type: Number, required: true},
	stageId: {type: Number, required: true},
	userId: {type: Number, required: true},
});

const RecommendOnlyDecisions = [
	pkp.const.DECISION_RECOMMEND_ACCEPT,
	pkp.const.DECISION_RECOMMEND_DECLINE,
	pkp.const.DECISION_RECOMMEND_PENDING_REVISIONS,
	pkp.const.DECISION_RECOMMEND_RESUBMIT,
];

const explicitelyShowRecommendationActions = ref(false);

const isDecidingEditorAssigned = computed(() => {
	return props.submission?.editorAssigned;
});

const showRecommendationActions = computed(() => {
	if (!isDecidingEditorAssigned.value) {
		return false;
	}

	if (explicitelyShowRecommendationActions.value) {
		return true;
	}

	if (!isLoading.value && !currentRecommendation.value) {
		return true;
	}

	return false;
});
function showActions() {
	explicitelyShowRecommendationActions.value = true;
}

function getRecommendationActions() {
	const actions = [];

	actions.push({
		label: t('editor.submission.recommend.revisions'),
		action: WorkflowActions.WORKFLOW_RECOMMEND_REVISION,
	});

	actions.push({
		label: t('editor.submission.recommend.accept'),
		action: DecisionActions.DECISION_RECOMMEND_ACCEPT,
	});

	actions.push({
		label: t('editor.submission.recommend.decline'),
		action: DecisionActions.DECISION_RECOMMEND_DECLINE,
	});

	return actions;
}

const actionItems = computed(() => {
	return getRecommendationActions();
});

const {apiUrl: recommendationApiUrl} = useUrl(
	// TODO improve url when the query params are available
	`submissions/${props.submission?.id}/decisions`,
);

const {
	data: recommendations,
	fetch: fetchRecommendations,
	isLoading,
} = useFetch(recommendationApiUrl);

fetchRecommendations();
watch(props, () => fetchRecommendations());

const currentRecommendation = computed(() => {
	if (!recommendations) {
		return null;
	}

	let recommendationsFromLatest = [...recommendations.value].reverse();

	const myLastRecommendation = recommendationsFromLatest.find(
		(recommendation) => {
			return (
				recommendation.editorId === props.userId &&
				recommendation.reviewRoundId === props.reviewRoundId &&
				RecommendOnlyDecisions.includes(recommendation.decision)
			);
		},
	);

	if (myLastRecommendation) {
		return {label: myLastRecommendation.label};
	} else {
		return null;
	}
});

const workflowStore = useWorkflowStore();
function handleAction(actionName) {
	workflowStore[actionName]({
		reviewRoundId: props.reviewRoundId,
		stageId: props.stageId,
	});
}
</script>
