<template>
	<div class="pkpTabs" :class="{'pkpTabs--side': isSideTabs}">
		<div class="pkpTabs__buttons" role="tablist" :aria-label="label">
			<button
				v-for="tab in tabs"
				:id="tab.id + '-button'"
				:key="tab.id"
				:ref="'button' + tab.id"
				:aria-selected="currentTab === tab.id"
				:aria-controls="tab.id"
				class="pkpTabs__button"
				role="tab"
				:tabindex="currentTab === tab.id ? '' : -1"
				@click="setCurrentTab(tab.id)"
				@keydown.end.prevent="setLastTab"
				@keydown.home.prevent="setFirstTab"
				@keydown.left.exact.prevent="setPreviousTab"
				@keydown.right.exact.prevent="setNextTab"
			>
				<template v-if="tab.icon">
					<Icon :icon="tab.icon" class="h-4 w-4"></Icon>
					<span class="-screenReader">{{ tab.label }}</span>
				</template>
				<template v-else>
					{{ tab.label }}
				</template>
				<template v-if="tab.badge">
					<Badge>{{ tab.badge }}</Badge>
				</template>
			</button>
		</div>
		<slot />
	</div>
</template>

<script>
import debounce from 'debounce';
import Icon from '@/components/Icon/Icon.vue';
import Badge from '@/components/Badge/Badge.vue';

export default {
	components: {Icon, Badge},
	props: {
		/** Select one of the tabs by default. Pass the tab's `id` prop. */
		defaultTab: {
			type: String,
			default() {
				return '';
			},
		},
		/** Displays the tabs on the side with content beside it when `true` */
		isSideTabs: {
			type: Boolean,
			default() {
				return false;
			},
		},
		/** Sets an `aria-label` for the tabs. Read the [accessible label](#accessible-label) section below. */
		label: {
			type: String,
			default() {
				return '';
			},
		},
		/** When `true`, changes to the current tab will modify the browser history so that the back button can be used. */
		trackHistory: {
			type: Boolean,
			default() {
				return false;
			},
		},
	},
	data() {
		return {
			currentTab: '',
			tabs: [],
		};
	},
	methods: {
		/**
		 * Set the current tab
		 *
		 * @param {String} tabId
		 */
		setCurrentTab(tabId) {
			this.currentTab = tabId;
			this.$nextTick(() => {
				$(this.$refs['button' + tabId]).focus();
				this.updateUrl();
			});
		},

		/**
		 * Set the current tab to the first tab in the list
		 *
		 * Keyboard: [Home]
		 */
		setFirstTab() {
			this.setCurrentTab(this.tabs[0].id);
		},

		/**
		 * Set the current tab to the last tab in the list
		 *
		 * Keyboard: [End]
		 */
		setLastTab() {
			this.setCurrentTab(this.tabs[this.tabs.length - 1].id);
		},

		/**
		 * Set the current tab to the next tab in the list.
		 * Jump to the start if the current tab is the last one.
		 *
		 * Keyboard: →
		 */
		setNextTab() {
			const i = this.tabs.findIndex((tab) => tab.id === this.currentTab);
			const tab = this.tabs[i + 1] || this.tabs[0];
			this.setCurrentTab(tab.id);
		},

		/**
		 * Set the current tab to the previous tab in the list.
		 * Jump to the end if the current tab is the first one.
		 *
		 * Keyboard: ←
		 */
		setPreviousTab() {
			const i = this.tabs.findIndex((tab) => tab.id === this.currentTab);
			const tab = this.tabs[i - 1] || this.tabs[this.tabs.length - 1];
			this.setCurrentTab(tab.id);
		},

		/**
		 * Update the URL hash when a tab has been opened
		 */
		updateUrl: debounce(function () {
			if (this.trackHistory) {
				const hash =
					this.$parent.$options.name === 'Tab'
						? '#' + this.$parent.id + '/' + this.currentTab
						: '#' + this.currentTab;
				if (hash !== window.location.hash) {
					const activeTab = this.tabs.find((tab) => tab.id === this.currentTab);
					window.history.pushState({}, activeTab.label, hash);
				}
			}
		}, 100),
	},
	watch: {
		/**
		 * Update the active tab when a new tab is selected
		 */
		currentTab(newVal, oldVal) {
			this.tabs.forEach((tab) => tab.isActive(tab.id === newVal));
		},
	},
	provide() {
		return {
			registerTab: (tab) => {
				this.tabs.push(tab);

				// Return an unregistration function for cleanup
				return () => {
					const index = this.tabs.findIndex((_tab) => _tab.id === tab.id);
					if (index > -1) {
						this.tabs.splice(index, 1);
					}
				};
			},
		};
	},

	mounted() {
		/**
		 * Set the tab to view when loaded
		 */
		this.currentTab = this.defaultTab || this.tabs[0].id;

		/**
		 * Listen for global 'open-tab' events and open the correct tab when called
		 */
		pkp.eventBus.$on('open-tab', (tabId) => {
			this.tabs.forEach((tab) => {
				if (tab.id === tabId) {
					this.setCurrentTab(tabId);
				}
			});
		});
	},
	unmounted() {
		pkp.eventBus.$off('open-tab');
	},
};
</script>

<style lang="less">
@import '../../styles/_import';

.pkpTabs {
	margin: 0;
	list-style: none;
	font-size: @font-sml;
}

.pkpTabs__button {
	position: relative;
	display: inline-block;
	padding: 0.8rem 1em;
	line-height: 1.4rem;
	border: @bg-border-light;
	border-top-left-radius: @radius;
	border-top-right-radius: @radius;
	background: @lift;
	text-decoration: none;
	font-weight: @bold;
	color: @primary;

	&:before {
		content: '';
		position: absolute;
		top: -1px;
		left: 50%;
		transform: translateX(-50%);
		width: 25%;
		height: 2px;
		background: transparent;
		transition: all 0.2s;
	}

	/* Ensure focus is visible at all times */
	&:focus {
		outline: 0;

		&:after {
			content: '';
			position: absolute;
			bottom: 0.375rem;
			width: 0.25rem;
			height: 0.25rem;
			left: 50%;
			transform: translateX(-50%);
			background-color: @text-light-rgba;
			border-radius: 0.125rem;
		}
	}

	+ .pkpTabs__button {
		margin-inline-start: 0.25rem;
	}

	.pkpBadge {
		margin-inline-start: 0.25em;
		margin-bottom: -3px; // Prevent change to tab height
		padding-left: 0.5em;
		padding-right: 0.5em;
		min-width: 2.25em; // Give single-digit badges a rounder shape
		border-color: @bg-border-color;
		background: @lift;
	}
}

.pkpTabs__button:focus:before,
.pkpTabs__button:hover:before,
.pkpTabs__button[aria-selected='true']:before {
	background: @primary-lift;
	width: 100%;
}

.pkpTabs__button[aria-selected='true'] {
	color: @text-light;
}

.pkpTab {
	position: relative;
	padding: 1rem;
	background: @lift;

	&:focus {
		z-index: 2;
	}
}

// Nested tabs
.pkpTabs .pkpTabs {
	margin-left: -1rem;
	margin-right: -1rem;
}

@media (min-width: 767px) {
	.pkpTabs__buttons {
		position: relative;
		top: 1px;
		z-index: 2;
	}

	.pkpTabs__button {
		border-color: transparent;
		background: transparent;
	}

	.pkpTabs__button[aria-selected='true'] {
		border: @bg-border-light;
		border-bottom-color: transparent;
		background: @lift;
	}

	.pkpTab {
		padding: 2rem;
		border: @bg-border-light;
		border-radius: @radius;
	}

	/* Nested tabs */
	.pkpTabs .pkpTabs {
		margin-left: -2rem;
		margin-right: -2rem;
		margin-bottom: -2rem;

		.pkpTabs__buttons {
			padding-left: 1rem;
			padding-right: 1rem;
		}

		.pkpTab {
			border: none;
			border-top: @bg-border-light;
		}
	}

	/* Side-tabs */
	.pkpTabs--side {
		display: grid;
		grid-template-columns: 192px auto;

		.pkpTabs__buttons {
			padding-left: 0;
			padding-right: 0;
			margin-bottom: 2rem;
			border-inline-end: @bg-border-light;
		}

		.pkpTabs__button {
			display: block;
			width: 100%;
			padding-right: 1rem;
			border-left: none;
			border-right: none;
			border-color: transparent;
			top: auto;
			right: -1px; // overlap right border
			text-align: inherit;

			&:before {
				content: '';
				position: absolute;
				top: 50%;
				left: -2px;
				transform: translateY(-50%);
				height: 25%;
				width: 2px;
				background: transparent;
				transition: all 0.2s;
			}

			&:focus {
				outline: 0;

				&:after {
					top: 50%;
					bottom: auto;
					left: 0.375rem;
					transform: translateY(-50%);
				}
			}

			+ .pkpTabs__button {
				margin-inline-end: 0;
			}
		}

		.pkpTabs__button:focus:before,
		.pkpTabs__button:hover:before,
		.pkpTabs__button[aria-selected='true']:before {
			background: @primary-lift;
			height: 100%;
			width: 2px;
		}

		.pkpTabs__button[aria-selected='true'] {
			border-color: @bg-border-color-light;
			border-left: none;
			border-right: none;
		}
	}

	.pkpTabs .pkpTabs--side .pkpTabs__buttons {
		padding-inline-end: 0;
	}

	.pkpTabs .pkpTabs--side .pkpTab {
		padding-top: 0;
		border-top: none;

		> .pkpForm {
			margin: 0;
			margin-inline-start: calc(-2rem - 1px);
			border: @bg-border-light;
		}

		// Fix off-by-one errors with the locale and side tab border lines
		&:nth-child(2) {
			.pkpForm > .pkpFormPages {
				margin-top: -1px;
			}
		}
	}

	/* Forms in tabs */
	.pkpTab > .pkpForm {
		margin: -2rem;
	}
}

[dir='rtl'] {
	@media (min-width: 767px) {
		.pkpTabs--side {
			.pkpTabs__button {
				left: -1px; // overlap right border
				right: auto;

				&:before {
					left: auto;
					right: -2px;
				}

				&:focus {
					&:after {
						left: auto;
						right: 0.375rem;
					}
				}
			}
		}
	}
}
</style>
