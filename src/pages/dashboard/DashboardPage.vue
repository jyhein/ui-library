<template>
	<div class="min-h-screentext-base-normal me-3 ms-5 text-base-normal">
		<div class="">
			<h2 class="flex items-center gap-4 py-6 text-5xl-bold">
				{{
					`${store.currentView.name} (${store.submissionsPagination.itemCount})`
				}}
			</h2>
			<div class="mt-2">
				<div class="flex justify-between">
					<PkpButton @click="store.openFiltersModal">
						{{ t('common.filter') }}
					</PkpButton>
					<div>
						<Search
							:search-phrase="store.searchPhrase"
							:search-label="t('editor.submission.search')"
							@search-phrase-changed="
								(...args) => store.setSearchPhrase(...args)
							"
						></Search>
					</div>
				</div>
			</div>
			<div class="mt-4">
				<DashboardActiveFilters
					:active-filters-list="store.filtersFormList"
					@clear-filters="store.clearFiltersForm"
					@remove-filter="store.clearFiltersFormField"
				/>
			</div>
			<div class="mt-4">
				<DashboardTable
					:items="store.submissions"
					:columns="store.columns"
					:sort-descriptor="store.sortDescriptor"
					:pagination="store.submissionsPagination"
					@sort-column="store.applySort"
					@set-page="store.setCurrentPage"
				/>
			</div>
		</div>
	</div>
</template>
<script setup>
import PkpButton from '@/components/Button/Button.vue';
import DashboardActiveFilters from './components/DashboardActiveFilters.vue';
import DashboardTable from './components/DashboardTable/DashboardTable.vue';
import Search from '@/components/Search/Search.vue';

import {useDashboardPageStore} from './dashboardPageStore';

const props = defineProps({
	dashboardPage: {
		required: true,
		type: String,
		validator: (prop) =>
			[
				'EDITORIAL_DASHBOARD',
				'MY_REVIEW_ASSIGNMENTS',
				'MY_SUBMISSIONS',
			].includes(prop),
	},
	/** Form for selection type of revision */
	selectRevisionDecisionForm: {type: Object, required: true},
	/** Form for selection type of revision for recommending editor */
	selectRevisionRecommendationForm: {type: Object, required: true},
	/** Form for Contributor manager */
	contributorForm: {type: Object, required: true},
	/** List of Views */
	views: {
		type: Array,
		required: true,
	},
	/** Filters form config  */
	filtersForm: {
		type: Object,
		required: true,
	},
	/** List of columns */
	columns: {
		type: Array,
		required: true,
	},
	/** How many submissions to show per page */
	countPerPage: {
		type: Number,
		required: true,
	},
	/** context settings {supportsCitations, } */
	publicationSettings: {
		type: Object,
		required: true,
	},
});

const store = useDashboardPageStore(props);
</script>

<style>
.pkp_page_dashboard .app__main {
	@apply bg-secondary p-0;
}
</style>
./components/DashboardActiveFilters.vue
