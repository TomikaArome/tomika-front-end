<template>
	<div class="tomika-popup" :class="{ borderless: $props.borderless }">
		<div v-if="!(noTitle || filteredChoices && filteredChoices.length)" class="tomika-popup-title-row">
			<h2 class="tomika-popup-title">{{ title }}</h2>
			<div class="spacer"></div>
			<div v-if="closeButtonAction" class="tomika-popup-close-button" @click="closeButtonAction">
				<font-awesome-icon icon="times"></font-awesome-icon>
			</div>
		</div>
		<div class="tomika-popup-container">
			<div class="tomika-popup-tabs">
				<div v-for="(tab, tabIndex) in tabs" :key="tabIndex"
					:class="{ 'tomika-popup-tab': !tab.spacer, 'tomika-popup-tab-spacer': tab.spacer,
					selected: selectedTab === tabIndex }" @click="clickTab(tabIndex)">
					<div v-if="!tab.spacer && (tab.image || tab.icon)" class="tomika-popup-tab-icon">
						<img v-if="tab.image" :src="tab.image">
						<font-awesome-icon :icon="tab.icon" v-if="tab.icon && !tab.image"></font-awesome-icon>
					</div>
					<div class="tomika-popup-tab-text" v-if="!tab.spacer && tab.text">
						<h2>{{ tab.text }}</h2>
						<div v-if="tab.subtext" class="subtext">{{ tab.subtext }}</div>
					</div>
				</div>
			</div>
			<div class="tomika-popup-content">
				<div class="tomika-popup-prompt" v-if="filteredChoices && filteredChoices.length && prompt">{{ prompt }}</div>
				<keep-alive v-else>
					<component :is="content"></component>
				</keep-alive>
				<div class="tomika-popup-choices" v-if="choices && this.choices.length">
					<button v-for="(choice, index) in filteredChoices" :key="index"
						@click="choice.buttonAction(closeButtonAction)">{{ choice.buttonText }}</button>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	/**
	 * This element merits explanation for its number of props:
	 *  - title: title of the popup, will display just above the content [REQUIRED]
	 *  - borderless: if set to true, the popup won't have borders and the corners won't be rounded. Useful for full
	 *      containers. Defaults to false
	 *  - noTitle: if set to true, the title bar won't be displayed
	 *  - closeButtonAction: function that will be triggered by the close button. If omitted, the close button will not
	 *      be displayed
	 *  - noScreenClose: if set to true, the popup won't close from clicking the screen (popup stack only)
	 *  - bigPopup: if set to true, the popup will be fixed in size and will spread across the entire screen on smaller
	 *      screens (popup stack only)
	 *  - ONE OF THE FOLLOWING:
	 *    - contentComponent: the component that will be displayed (no tabs solution)
	 *    OR
	 *    - tabs: an array of objects with:
	 *      - contentComponent: the component that will be displayed when the tab is selected
	 *      - text: the text describing what the tab is about
	 *      - subtext: some smaller text underneath
	 *    OR
	 *    - prompt: a question or prompt to be displayed in the popup
	 *    - choices: an array of objects representing the choices for the prompt, with:
	 *      - buttonText: the text inside the button
	 *      - buttonClass: a class name to style the button
	 *      - buttonAction: a callback function for when the button is activated, the first parameter of this function
	 *          is the function to close the popup
	 */

	// Import dependencies
	import Vue from 'vue';
	import { library } from '@fortawesome/fontawesome-svg-core';
	import { faTimes } from '@fortawesome/free-solid-svg-icons';
	import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';

	// Font awesome
	library.add(faTimes);
	Vue.component('font-awesome-icon', FontAwesomeIcon);

	export default {
		name: "tomika-popup",
		data() {
			return {
				selectedTab: 0
			}
		},
		props: {
			title: String,
			contentComponent: Object,
			borderless: { type: Boolean, default: false },
			noTitle: { type: Boolean, default: false },
			closeButtonAction: Function,
			tabs: Array,
			prompt: String,
			choices: Array
		},
		computed: {
			content() {
				if (this.tabs.length > 0) {
					return this.tabs[this.selectedTab].contentComponent || this.contentComponent || 'div';
				}
				return this.contentComponent || 'div';
			},
			filteredChoices() {
				return this.choices ? this.choices.filter((choice) => { return typeof choice.buttonAction === 'function' }) : [];
			}
		},
		methods: {
			clickTab(tabIndex) {
				if (!this.tabs[tabIndex].spacer) {
					this.selectedTab = tabIndex;
				}
			}
		}
	}
</script>

<style scoped>
	.tomika-popup {
		display: flex;
		position: relative;
		flex-direction: column;
		background-color: hsl(0,0%,10%);
	}
	.tomika-popup:not(.borderless) {
		border: 1px solid hsl(180,25%,30%);
		border-radius: 16px;
		overflow: hidden;
	}
	.tomika-popup-container {
		display: flex;
		position: relative;
		flex-grow: 1;
		flex-direction: row;
		height: calc(100% - 40px);
	}
	.tomika-popup-title-row {
		display: flex;
		flex-direction: row;
		align-items: center;
		padding: 0 0 0 20px;
		height: 40px;
		background-color: hsl(0,0%,5%);
	}
	.tomika-popup-title {
		margin: 0;
		font-size: 20px;
		text-shadow: -2px -2px hsla(0,0%,0%,1);
	}
	.tomika-popup-close-button {
		width: 25px;
		height: 25px;
		margin-right: 6px;
		border-radius: 100%;
		display: flex;
		justify-content: center;
		align-items: center;
		color: hsl(0,0%,80%);
		cursor: pointer;
		user-select: none;
		transition: background-color 150ms;
	}
	.tomika-popup-close-button:hover {
		background-color: hsl(0,25%,30%);
		color: hsl(0,25%,80%);
	}
	.tomika-popup-tabs {
		display: flex;
		flex-direction: column;
		background-color: hsl(0,0%,2%);
		color: hsl(0,0%,80%);
	}
	.tomika-popup-tab-spacer:not(:first-child):not(:last-child) {
		padding-top: 16px;
	}
	.tomika-popup-tab {
		display: flex;
		align-items: center;
		cursor: pointer;
		user-select: none;
		height: 50px;
		min-width: 25px;
		transition: background-color 150ms;
		box-sizing: border-box;
		padding: 0 8px 0 8px;
		background-color: hsl(0,0%,7%);
	}
	.tomika-popup-tab:hover, .selected {
		background-color: hsl(0,0%,10%);
	}
	.tomika-popup-tab-icon {
		position: relative;
		display: flex;
		justify-content: center;
		align-items: center;
		height: 38px;
		width: 38px;
		font-size: 24px;
		transition: background-color 150ms;
	}
	.tomika-popup-tab-icon img {
		width: 100%;
		border-radius: 100%;
	}
	.tomika-popup-tab-text {
		margin: 0 8px;
		white-space: nowrap;
	}
	.tomika-popup-tab-text > h2 {
		margin: 0;
		font-size: 16px;
		line-height: 20px;
	}
	.tomika-popup-tab-text > .subtext {
		color: hsl(0,0%,50%);
		font-size: 12px;
	}
	.tomika-popup-content {
		display: flex;
		flex-direction: column;
		flex-grow: 1;
		min-width: 200px;
		min-height: 100px;
		overflow-y: scroll;
		overflow-x: hidden;
		scrollbar-width: thin;
		scrollbar-color: hsl(0,0%,20%) hsl(0,0%,10%);
	}
	.tomika-popup-content::-webkit-scrollbar {
		width: 10px;
		background-color: hsl(0,0%,10%);
	}
	.tomika-popup-content::-webkit-scrollbar-thumb {
		background-color: hsl(0,0%,20%);
		outline: 1px solid slategrey;
		border-radius: 5px;
		border: 2px solid hsl(0,0%,10%);
		transition: background-color 150ms;
	}
	.tomika-popup-content::-webkit-scrollbar-thumb:hover {
		background-color: hsl(0,0%,30%);
	}
	.tomika-popup-content > :first-child {
		flex-grow: 1;
	}
	.tomika-popup-prompt {
		display: flex;
		justify-content: center;
		align-items: center;
		text-align: center;
		font-size: 16px;
		padding: 12px 24px;
		max-width: 400px;
	}
	.tomika-popup-choices {
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: center;
		padding: 8px 0;
	}
	.tomika-popup-choices > button { margin: 0 4px }
	.tomika-popup-choices > button:first-child { margin-left: 16px; }
	.tomika-popup-choices > button:last-child { margin-right: 16px; }
</style>