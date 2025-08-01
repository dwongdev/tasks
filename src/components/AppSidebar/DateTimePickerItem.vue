<!--
Nextcloud - Tasks

@author Raimund Schlüßler
@copyright 2021 Raimund Schlüßler <raimund.schluessler@mailbox.org>

This library is free software; you can redistribute it and/or
modify it under the terms of the GNU AFFERO GENERAL PUBLIC LICENSE
License as published by the Free Software Foundation; either
version 3 of the License, or any later version.

This library is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU AFFERO GENERAL PUBLIC LICENSE for more details.

You should have received a copy of the GNU Affero General Public
License along with this library. If not, see <http://www.gnu.org/licenses/>.

-->

<template>
	<div v-click-outside="checkOutsideClick"
		:class="{
			'property__item--clearable': date.isValid() && !readOnly,
			'property__item--valid': isValid,
			'property__item--overdue': isOverdue,
			'property__item--readonly': readOnly
		}"
		class="property__item">
		<div class="item__content" @click="setEditing(true)">
			<span class="content__icon">
				<slot name="icon" />
			</span>
			<span v-show="!editing" class="content__name">
				{{ propertyString }}
			</span>
			<div v-if="editing" class="content__input">
				<NcDateTimePicker :model-value="newValue"
					:clearable="false"
					:append-to-body="true"
					:show-week-number="true"
					type="date"
					:placeholder="t('tasks', 'Set date')"
					class="date"
					@update:model-value="setDate" />
				<NcDateTimePicker v-if="!allDay"
					:model-value="newValue"
					:clearable="false"
					:append-to-body="true"
					:minute-step="30"
					type="time"
					:placeholder="t('tasks', 'Set time')"
					class="time"
					@update:model-value="setTime" />
			</div>
		</div>
		<div class="item__actions">
			<NcActions v-show="editing" class="actions__set">
				<NcActionButton @click="setValue()">
					<template #icon>
						<Check :size="20" />
					</template>
					{{ t('tasks', 'Set date') }}
				</NcActionButton>
			</NcActions><NcActions v-show="editing" class="actions__clear">
				<NcActionButton @click="clearValue">
					<template #icon>
						<Delete :size="20" />
					</template>
					{{ t('tasks', 'Delete date') }}
				</NcActionButton>
			</NcActions>
		</div>
	</div>
</template>

<script>
import editableItem from '../../mixins/editableItem.js'
import { overdue } from '../../store/storeHelper.js'

import { translate as t } from '@nextcloud/l10n'
import NcDateTimePicker from '@nextcloud/vue/components/NcDateTimePicker'

export default {
	name: 'DateTimePickerItem',
	components: {
		NcDateTimePicker,
	},
	mixins: [editableItem],
	props: {
		/**
		 * The current date
		 */
		date: {
			type: Object,
			default: null,
		},
		/**
		 * The current value
		 */
		value: {
			type: Date,
			default: null,
		},
		/**
		 * Whether the date is all day
		 */
		allDay: {
			type: Boolean,
			default: false,
		},
		/**
		 * Whether the date can be considered 'overdue'
		 */
		checkOverdue: {
			type: Boolean,
			default: true,
		},
	},
	computed: {
		isValid() {
			return this.date.isValid()
		},
		isOverdue() {
			return this.checkOverdue && overdue(this.date)
		},
	},
	methods: {
		t,

		/**
		 * Checks if the click originated from the datepicker
		 * and sets the value if not.
		 *
		 * @param {object} $event The click event
		 */
		checkOutsideClick($event) {
			/**
			 * If the click originates from the datepicker, we do nothing.
			 */
			if ($event.target.closest('.dp__outer_menu_wrap')) {
				return
			}
			this.setValue()
		},
		/**
		 * When selecting a new date with the datepicker, the time is always set to 00:00.
		 * So we have to get the time from the previous date object.
		 *
		 * @param {Date} date The new date (year, month & day)
		 */
		setDate(date) {
			date.setHours(this.newValue.getHours(), this.newValue.getMinutes())
			this.newValue = date
		},
		/**
		 * When selecting a new time with the keyboard, the date is set to the current day.
		 * So we only get hours and minutes from the new date.
		 *
		 * @param {Date} date The new date (hours & minutes)
		 */
		setTime(date) {
			/**
			 * Simply mutating newValue doesn't make vue pick up the changes,
			 * so we create a new object, mutate it and assign it to newValue.
			 */
			const newDate = new Date(this.newValue.getTime())
			newDate.setHours(date.getHours(), date.getMinutes())
			this.newValue = newDate
		},
	},
}
</script>

<style lang="scss" scoped>
$red: #b3312d;
$blue: #4271a6;

.property__item {
	border-bottom: 1px solid var(--color-border);
	padding: 0 6px;
	position: relative;
	margin-bottom: 0;
	width: 100%;
	color: var(--color-text-lighter);
	display: flex;

	&:first-child {
		border-top: 1px solid var(--color-border);
	}

	& * {
		cursor: pointer;
	}

	.item {
		&__content {
			display: flex;
			line-height: var(--default-clickable-area);
			min-width: 0;
			flex-grow: 1;
			gap: 0 4px;

			.content {
				&__icon {
					display: flex;
					height: var(--default-clickable-area);
					width: var(--default-clickable-area);
					min-width: var(--default-clickable-area);
					justify-content: center;

					.material-design-icon__svg {
						vertical-align: middle;
					}
				}

				&__name {
					flex-grow: 1;
					padding-right: 14px;
					overflow: hidden;
					text-overflow: ellipsis;
					white-space: nowrap;
				}

				&__input {
					display: flex;
					flex-grow: 1;
					flex-wrap: wrap;
					grid-gap: var(--default-grid-baseline);

					.vue-date-time-picker__wrapper {
						width: auto;
						&.date {
							min-width: 100px;
							flex-shrink: 1;
							flex-basis: 100px;
							flex-grow: 3;
						}
						&.time {
							min-width: 65px;
							flex-shrink: 2;
							flex-basis: 65px;
							flex-grow: 2;
						}
						:deep(input) {
							margin: 0;
						}
					}
				}
			}
		}

		&__actions {
			display: flex;
			flex-wrap: nowrap;
		}
	}

	&--valid {
		color: $blue;
	}

	&--overdue {
		color: $red;
	}

	&--readonly * {
		cursor: default;
	}

	&--clearable:hover .actions__clear {
		display: inline-block !important;
	}
}
</style>
