<script>
import Page from './Page.vue';
import DateRange from '@/components/DateRange/DateRange.vue';
import PkpFilter from '@/components/Filter/Filter.vue';
import Pagination from '@/components/Pagination/Pagination.vue';
import PkpHeader from '@/components/Header/Header.vue';
import PkpTable from '@/components/Table/Table.vue';
import TableCell from '@/components/Table/TableCell.vue';
import TableColumn from '@/components/Table/TableColumn.vue';
import TableHeader from '@/components/Table/TableHeader.vue';
import TableBody from '@/components/Table/TableBody.vue';
import TableRow from '@/components/Table/TableRow.vue';
import ajaxError from '@/mixins/ajaxError';

export default {
	name: 'StatsPage',
	components: {
		DateRange,
		PkpFilter,
		Pagination,
		PkpHeader,
		PkpTable,
		TableCell,
		TableColumn,
		TableHeader,
		TableBody,
		TableRow,
	},
	extends: Page,
	mixins: [ajaxError],
	data() {
		return {
			/** URL to the REST API endpoint to retrieve stats. */
			apiUrl: '',
			/** An array of objects representing columns in the table. See [Table](#/component/Table). */
			tableColumns: [],
			/** Current start date in the format `YYYY-MM-DD`. */
			dateStart: '',
			/** Current start date in the format `YYYY-MM-DD`. */
			dateEnd: '',
			/** Current start date in the format `YYYY-MM-DD`. */
			dateEndMax: '',
			/** Array of objects representing preset date range options, such as the last 90 days. */
			dateRangeOptions: [],
			/** Array of filters. */
			filters: [],
			activeFilters: {},
			isSidebarVisible: false,
			isLoadingItems: false,
			/** A label to display the count of the current page. */
			itemsOfTotalLabel: '',
			latestItemsGetRequest: '',
		};
	},
	computed: {
		/**
		 * Add a class to the sidebar when it is visible
		 *
		 * @return Array
		 */
		sidebarClasses() {
			let classes = [];
			if (this.isSidebarVisible) {
				classes.push('-isVisible');
			}
			return classes;
		},
	},
	watch: {
		activeFilters(newVal, oldVal) {
			if (newVal === oldVal) {
				return;
			}
			this.offset = 0;
			this.get();
		},
		dateEnd(newVal, oldVal) {
			if (newVal === oldVal) {
				return;
			}
			this.offset = 0;
			this.get();
		},
		dateStart(newVal, oldVal) {
			if (newVal === oldVal) {
				return;
			}
			this.offset = 0;
			this.get();
		},
		isSidebarVisible(newVal, oldVal) {
			if (newVal === oldVal) {
				return;
			}
			this.setSidebarFocus();

			// move focus into the sidebar when it is made visible
			if (newVal) {
				this.$nextTick(() => {
					if (newVal && Object.keys(this.$refs).includes('sidebar')) {
						this.setFocusIn(this.$refs.sidebar);
					}
				});
			}
		},
	},
	mounted() {
		/**
		 * Set the initial tabindex attributes in the sidebar
		 */
		this.setSidebarFocus();
	},
	methods: {
		/**
		 * Check if a filter is currently active
		 *
		 * @param {String} param
		 * @param {mixed} value
		 * @return {Boolean}
		 */
		isFilterActive: function (param, value) {
			if (!Object.keys(this.activeFilters).includes(param)) {
				return false;
			}
			if (Array.isArray(this.activeFilters[param])) {
				return this.activeFilters[param].includes(value);
			}
			return this.activeFilters[param] === value;
		},

		/**
		 * Set the date range
		 *
		 * @param string dateStart
		 * @param string dateEnd
		 */
		setDateRange(dateStart, dateEnd) {
			this.dateStart = dateStart;
			this.dateEnd = dateEnd;
		},

		/**
		 * Toggle filter visibility
		 */
		toggleSidebar() {
			this.isSidebarVisible = !this.isSidebarVisible;
			if (!this.isSidebarVisible) {
				this.activeFilters = {};
				this.get();
			}
		},

		/**
		 * Add a filter
		 *
		 * Adds the value to the array of existing values for
		 * this param. Use this method for filters which accept
		 * more than one value for the param. For example, a
		 * list of valid stageIds.
		 *
		 * @param {String} param
		 * @param {mixed} value
		 */
		addFilter: function (param, value) {
			if (this.isFilterActive(param, value)) {
				return;
			}
			let filters = {...this.activeFilters};
			if (filters[param] === undefined) {
				filters[param] = [];
			}
			filters[param].push(value);
			this.activeFilters = filters;
			this.get();
		},

		/**
		 * Remove a filter
		 *
		 * Removes one value from the array of values for this
		 * param. Use this method for filters that use addFilter.
		 *
		 * @param {String} param
		 * @param {mixed} value
		 */
		removeFilter: function (param, value) {
			if (this.activeFilters[param] === undefined) {
				return;
			}
			let filters = {...this.activeFilters};
			filters[param] = filters[param].filter(
				(filterVal) => filterVal !== value,
			);
			this.activeFilters = filters;
			this.get();
		},

		/**
		 * Get the number of days betweeen two dates
		 *
		 * This could probably be moved into a general function somewhere.
		 *
		 * @param Date dateStart
		 * @param Date dateEnd
		 * @return Number
		 */
		getDaysBetween(dateStart, dateEnd) {
			const millisecondsPerDay = 24 * 60 * 60 * 1000;

			// Handle timezone offsets if date goes over Daylight Savings Time
			// See: https://stackoverflow.com/a/11252167
			dateStart.setMinutes(
				dateStart.getMinutes() - dateStart.getTimezoneOffset(),
			);
			dateEnd.setMinutes(dateEnd.getMinutes() - dateEnd.getTimezoneOffset());

			return (dateEnd - dateStart) / millisecondsPerDay;
		},

		/**
		 * Set the tabindex on items in the sidebar to allow/prevent
		 * them from accepting focus
		 *
		 * @param {Boolean} enable
		 */
		setSidebarFocus() {
			if (!this.$refs.sidebar) {
				return;
			}
			const tabIndex = this.isSidebarVisible ? 0 : -1;
			this.$refs.sidebar
				.querySelectorAll('button, [href], input, select, textarea')
				.forEach((el) => (el.tabIndex = tabIndex));
		},
	},
};
</script>

<style lang="less">
@import '../../styles/_import';

.pkpStats > .pkpHeader {
	padding: 0.5rem 0;
}

.pkpStats__graph {
	margin-bottom: 1rem;
}

.pkpStats__content {
	margin-inline-start: 0;
}

.pkpStats__sidebar {
	/** Intention here seems to be hiding from screen, but keeping for screen reader
		Not sure if thats best practice in this case, but for now using tailwindcss classes,
		which work with RTL. 
	*/

	@apply sr-only;

	+ .pkpStats__content {
		float: right;
		width: 100%;
		transition: width 0.2s;
	}

	.pkpHeader,
	.pkpFilter {
		margin-inline-start: -1rem;
	}

	.pkpStats__filterSet:first-child .pkpHeader {
		padding-top: 0;
	}
}

.pkpStats__sidebar.-isVisible {
	@apply not-sr-only;
	float: left;
	position: relative;
	left: 0;
	width: 25%;
	opacity: 1;
	transition:
		opacity 0.2s ease-in-out 0.2s,
		left 0s ease-in-out 0.1s,
		right 0s ease-in-out 0.1s,
		width 0.2s ease-in-out 0s;

	+ .pkpStats__content {
		width: 75%;
	}
}

.pkpStats__sidebar .pkpHeader {
	padding-top: 1.5rem;
	padding-bottom: 0;
	padding-inline-start: 1rem;
	padding-inline-end: 0.5rem;
}

.pkpStats__sidebar > .pkpHeader:first-child {
	padding: 0.5rem;
	padding-inline-start: 1rem;
}

.pkpStats__sidebar .pkpHeader__title > h1,
.pkpStats__sidebar .pkpHeader__title > h2,
.pkpStats__sidebar .pkpHeader__title > h3,
.pkpStats__sidebar .pkpHeader__title > h4,
.pkpStats__sidebar .pkpHeader__title > h5,
.pkpStats__sidebar .pkpHeader__title > h6,
.pkpStats__sidebar .pkpHeader__title {
	font-size: @font-sml;
	font-weight: @bold;
	line-height: 1.5em;
}

.pkpStats__filterSet {
	margin: 1rem 0;
}
.pkpStats__panel {
	padding: 1rem;
	background: #fff;
	border-radius: @radius;

	> .pkpHeader {
		margin-top: -1rem;
		padding: 0.5rem 0;
	}
}

.pkpStats__itemsOfTotal {
	font-size: @font-tiny;
}

.pkpStats__titleSearch {
	display: inline-block;
	float: none;
	margin-top: 0;
	margin-inline-start: 1rem;
	max-width: 20em;

	.pkpSearch__input {
		font-size: @font-tiny;
		line-height: 2.5em;
		padding-inline-start: 2.5rem;
	}
}

.pkpStats__itemLink {
	color: @text;
	text-decoration: none;

	&:hover,
	&:focus {
		color: @primary;
		text-decoration: underline;
		outline: 5px solid transparent;
	}
}

.pkpStats__itemAuthors {
	font-weight: @bold;
}

.pkpStats__noRecords {
	padding: 2rem 1rem;
	border: @grid-border;
	border-top: none;
	font-size: @font-sml;
	text-align: center;
	color: @text-light;
}

.pkpStats__graph {
	background: @bg-anchor;
	color: #fff;
	border-radius: @radius;

	.chartjs-render-monitor {
		border-radius: @radius;
	}
}

.pkpStats__graphHeader {
	padding: 1rem;
}

.pkpStats__graphSelectors {
	display: flex;
	align-items: center;
}

.pkpStats__graphSelector {
	display: flex;

	.pkpButton {
		position: relative;
		z-index: 1;
		background: transparent;
		border: 1px solid #437b96;
		box-shadow: 0 1px 0 #000;
		font-size: @font-tiny;
		line-height: 2em;
		color: #fff;

		&:before {
			content: '';
			position: relative;
			display: inline-block;
			width: 0.75em;
			height: 0.75em;
			margin-inline-end: 0.25em;
			border: 1px solid #fff;
			border-radius: 50%;
		}

		&:first-child {
			position: relative;
			left: 1px;
			border-top-right-radius: 0;
			border-bottom-right-radius: 0;
		}

		&:last-child {
			border-top-left-radius: 0;
			border-bottom-left-radius: 0;
		}

		&:hover,
		&:focus {
			color: #fff;
			border-color: #fff;
			z-index: 2;
		}

		&[aria-pressed='true'] {
			background: @primary;

			&:before {
				background: #fff;
				box-shadow: inset 0 0 0 1px @primary;
			}
		}

		&[disabled] {
			background: transparent;
			opacity: 0.5;
		}
	}
}

.pkpStats__graphSelector--timelineInterval {
	margin-inline-start: auto;
}

// Fade the graph and table when loading
.pkpStats__graph {
	position: relative;
}

.pkpStats__loadingCover {
	position: absolute;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	background: rgba(0, 0, 0, 0.5);
	z-index: 1;

	> .pkpSpinner {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);

		&:before {
			width: 100px;
			height: 100px;
			border-top-color: #fff;
			border-left-color: #fff;
			animation-duration: 0.8s;
		}
	}
}

.pkpStats__table .-isLoading tbody {
	opacity: 0.5;
}

[dir='rtl'] {
	.pkpStats__sidebar {
		+ .pkpStats__content {
			float: left;
		}
	}

	.pkpStats__sidebar.-isVisible {
		float: right;
		left: auto;
		right: 0;
	}

	.pkpStats__graphSelector {
		border-radius: 50%;

		&:first-child {
			left: auto;
			right: 1px;
			border-top-left-radius: 0;
			border-bottom-left-radius: 0;
		}

		&:last-child {
			border-top-right-radius: 0;
			border-bottom-right-radius: 0;
		}
	}

	.pkpStats__loadingCover {
		left: auto;
		right: 0;
	}
}
</style>
