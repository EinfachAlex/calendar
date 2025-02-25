<!--
  - @copyright Copyright (c) 2019 Georg Ehrke <oc.list@georgehrke.com>
  - @author Georg Ehrke <oc.list@georgehrke.com>
  -
  - @license AGPL-3.0-or-later
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<template>
	<AppNavigationItem :title="sharee.displayName">
		<template slot="icon">
			<AccountMultiple v-if="sharee.isGroup"
				:size="18"
				decorative
				class="avatar" />
			<IconCircle v-else-if="sharee.isCircle" />
			<Avatar v-else :user="sharee.id" :display-name="sharee.displayName" />
		</template>

		<template slot="counter">
			<ActionCheckbox :disabled="updatingSharee"
				:checked="sharee.writeable"
				@update:checked="updatePermission">
				{{ $t('calendar', 'can edit') }}
			</ActionCheckbox>
		</template>

		<template slot="actions">
			<ActionButton :disabled="updatingSharee"
				@click.prevent.stop="unshare">
				<template #icon>
					<Delete :size="20" decorative />
				</template>
				{{ $t('calendar', 'Unshare with {displayName}', { displayName: sharee.displayName }) }}
			</ActionButton>
		</template>
	</AppNavigationItem>
</template>

<script>
import ActionButton from '@nextcloud/vue/dist/Components/NcActionButton.js'
import ActionCheckbox from '@nextcloud/vue/dist/Components/NcActionCheckbox.js'
import AppNavigationItem from '@nextcloud/vue/dist/Components/NcAppNavigationItem.js'
import Avatar from '@nextcloud/vue/dist/Components/NcAvatar.js'
import {
	showInfo,
} from '@nextcloud/dialogs'

import AccountMultiple from 'vue-material-design-icons/AccountMultiple.vue'
import IconCircle from '../../Icons/IconCircles.vue'
import Delete from 'vue-material-design-icons/Delete.vue'

export default {
	name: 'CalendarListItemSharingShareItem',
	components: {
		ActionButton,
		ActionCheckbox,
		AppNavigationItem,
		Avatar,
	  IconCircle,
		AccountMultiple,
		Delete,
	},
	props: {
		calendar: {
			type: Object,
			required: true,
		},
		sharee: {
			type: Object,
			required: true,
		},
	},
	data() {
		return {
			updatingSharee: false,
		}
	},
	computed: {
		uid() {
			return this._uid
		},
	},
	methods: {
		/**
		 * Unshares the calendar from the given sharee
		 *
		 * @return {Promise<void>}
		 */
		async unshare() {
			this.updatingSharee = true
			try {
				await this.$store.dispatch('unshareCalendar', {
					calendar: this.calendar,
					uri: this.sharee.uri,
				})
				this.updatingSharee = false
			} catch (error) {
				console.error(error)
				showInfo(this.$t('calendar', 'An error occurred, unable to change the unshare the calendar.'))

				this.updatingSharee = false
			}
		},
		/**
		 * Toggles the write-permission of the share
		 *
		 * @return {Promise<void>}
		 */
		async updatePermission() {
			this.updatingSharee = true
			try {
				await this.$store.dispatch('toggleCalendarShareWritable', {
					calendar: this.calendar,
					uri: this.sharee.uri,
				})
				this.updatingSharee = false
			} catch (error) {
				console.error(error)
				showInfo(this.$t('calendar', 'An error occurred, unable to change the permission of the share.'))

				this.updatingSharee = false
			}
		},
	},
}
</script>
