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
	<AppNavigationItem :title="$t('calendar', 'Share link')"
		:menu-open.sync="menuOpen">
		<template slot="icon">
			<LinkVariant :class="{published: isPublished}"
				:size="18"
				decorative
				class="avatar" />
		</template>

		<template v-if="!isPublished" slot="actions">
			<ActionButton v-if="!publishingCalendar"
				key="publish"
				@click.prevent.stop="publishCalendar">
				<template #icon>
					<Plus :size="20" decorative />
				</template>
				{{ $t('calendar', 'Publish calendar') }}
			</ActionButton>
			<ActionButton v-else
				key="publishing"
				icon="icon-loading-small"
				:disabled="true">
				{{ $t('calendar', 'Publishing calendar') }}
			</ActionButton>
		</template>

		<template v-if="isPublished" slot="counter">
			<Actions>
				<ActionButton @click.prevent.stop="copyPublicLink">
					<template #icon>
						<ClipboardArrowLeftOutline :size="20" decorative />
					</template>
					{{ $t('calendar', 'Copy public link') }}
				</ActionButton>
			</Actions>
		</template>
		<template v-if="isPublished" slot="actions">
			<ActionButton v-if="showEMailLabel"
				@click.prevent.stop="openEMailLinkInput">
				<template #icon>
					<Email :size="20" decorative />
				</template>
				{{ $t('calendar', 'Send link to calendar via email') }}
			</ActionButton>
			<ActionInput v-if="showEMailInput"
				@submit.prevent.stop="sendLinkViaEMail">
				<template #icon>
					<Email :size="20" decorative />
				</template>
				{{ $t('calendar', 'Enter one address') }}
			</ActionInput>
			<ActionText v-if="showEMailSending"
				icon="icon-loading-small">
				<!-- eslint-disable-next-line no-irregular-whitespace -->
				{{ $t('calendar', 'Sending email …') }}
			</ActionText>

			<ActionButton v-if="showCopySubscriptionLinkLabel"
				@click.prevent.stop="copySubscriptionLink">
				<template #icon>
					<CalendarBlank :size="20" decorative />
				</template>
				{{ $t('calendar', 'Copy subscription link') }}
			</ActionButton>
			<ActionText v-if="showCopySubscriptionLinkSpinner"
				icon="icon-loading-small">
				<!-- eslint-disable-next-line no-irregular-whitespace -->
				{{ $t('calendar', 'Copying link …') }}
			</ActionText>
			<ActionText v-if="showCopySubscriptionLinkSuccess">
				<template #icon>
					<CalendarBlank :size="20" decorative />
				</template>
				{{ $t('calendar', 'Copied link') }}
			</ActionText>
			<ActionText v-if="showCopySubscriptionLinkError">
				<template #icon>
					<CalendarBlank :size="20" decorative />
				</template>
				{{ $t('calendar', 'Could not copy link') }}
			</ActionText>

			<ActionButton v-if="showCopyEmbedCodeLinkLabel"
				@click.prevent.stop="copyEmbedCode">
				<template #icon>
					<CodeBrackets :size="20" decorative />
				</template>
				{{ $t('calendar', 'Copy embedding code') }}
			</ActionButton>
			<ActionText v-if="showCopyEmbedCodeLinkSpinner"
				icon="icon-loading-small">
				<!-- eslint-disable-next-line no-irregular-whitespace -->
				{{ $t('calendar', 'Copying code …') }}
			</ActionText>
			<ActionText v-if="showCopyEmbedCodeLinkSuccess">
				<template #icon>
					<CodeBrackets :size="20" decorative />
				</template>
				{{ $t('calendar', 'Copied code') }}
			</ActionText>
			<ActionText v-if="showCopyEmbedCodeLinkError">
				<template #icon>
					<CodeBrackets :size="20" decorative />
				</template>
				{{ $t('calendar', 'Could not copy code') }}
			</ActionText>

			<ActionButton v-if="!unpublishingCalendar"
				@click.prevent.stop="unpublishCalendar">
				<template #icon>
					<Delete :size="20" decorative />
				</template>
				{{ $t('calendar', 'Delete share link') }}
			</ActionButton>
			<ActionText v-if="unpublishingCalendar"
				icon="icon-loading-small">
				<!-- eslint-disable-next-line no-irregular-whitespace -->
				{{ $t('calendar', 'Deleting share link …') }}
			</ActionText>
		</template>
	</AppNavigationItem>
</template>

<script>
import Actions from '@nextcloud/vue/dist/Components/NcActions.js'
import ActionButton from '@nextcloud/vue/dist/Components/NcActionButton.js'
import ActionInput from '@nextcloud/vue/dist/Components/NcActionInput.js'
import ActionText from '@nextcloud/vue/dist/Components/NcActionText.js'
import AppNavigationItem from '@nextcloud/vue/dist/Components/NcAppNavigationItem.js'
import ClickOutside from 'vue-click-outside'
import {
	generateRemoteUrl,
	generateUrl,
	linkTo,
} from '@nextcloud/router'
import {
	showSuccess,
	showError,
} from '@nextcloud/dialogs'
import HttpClient from '@nextcloud/axios'

import CalendarBlank from 'vue-material-design-icons/CalendarBlank.vue'
import ClipboardArrowLeftOutline from 'vue-material-design-icons/ClipboardArrowLeftOutline.vue'
import CodeBrackets from 'vue-material-design-icons/CodeBrackets.vue'
import Delete from 'vue-material-design-icons/Delete.vue'
import Email from 'vue-material-design-icons/Email.vue'
import LinkVariant from 'vue-material-design-icons/LinkVariant.vue'
import Plus from 'vue-material-design-icons/Plus.vue'

export default {
	name: 'CalendarListItemSharingPublishItem',
	components: {
		Actions,
		ActionButton,
		ActionInput,
		ActionText,
		AppNavigationItem,
		CalendarBlank,
		ClipboardArrowLeftOutline,
		CodeBrackets,
		Delete,
		Email,
		LinkVariant,
		Plus,
	},
	directives: {
		ClickOutside,
	},
	props: {
		calendar: {
			type: Object,
			required: true,
		},
	},
	data() {
		return {
			// is the calendar being published right now?
			publishingCalendar: false,
			// Send email action
			showEMailLabel: true,
			showEMailInput: false,
			showEMailSending: false,
			// copy public link:
			showCopyPublicLinkLabel: true,
			showCopyPublicLinkSpinner: false,
			// copy subscription link:
			showCopySubscriptionLinkLabel: true,
			showCopySubscriptionLinkSpinner: false,
			showCopySubscriptionLinkSuccess: false,
			showCopySubscriptionLinkError: false,
			// copy embed code:
			showCopyEmbedCodeLinkLabel: true,
			showCopyEmbedCodeLinkSpinner: false,
			showCopyEmbedCodeLinkSuccess: false,
			showCopyEmbedCodeLinkError: false,
			// delete public link
			unpublishingCalendar: false,
			// Status of actions menu:
			menuOpen: false,
		}
	},
	computed: {
		isPublished() {
			return this.calendar.publishURL !== null
		},
	},
	methods: {
		async publishCalendar() {
			this.publishingCalendar = true

			try {
				await this.$store.dispatch('publishCalendar', { calendar: this.calendar })
			} catch (error) {
				console.debug(error)
				showError(this.$t('calendar', 'An error occurred, unable to publish calendar.'))
			} finally {
				this.publishingCalendar = false
			}
		},
		openEMailLinkInput() {
			this.showEMailLabel = false
			this.showEMailInput = true
			this.showEMailSending = false
		},
		async sendLinkViaEMail(event) {
			this.showEMailLabel = false
			this.showEMailInput = false
			this.showEMailSending = true

			const emailAddress = event.target.querySelector('input[type=text]').value
			try {
				const url = [
					linkTo('calendar', 'index.php'),
					'v1/public/sendmail',
				].join('/')
				await HttpClient.post(url, {
					recipient: emailAddress,
					token: this.calendar.publishURL.split('/').slice(-1)[0],
					calendarName: this.calendar.displayName,
				})
			} catch (error) {
				console.error(error)
				showError(this.$t('calendar', 'An error occurred, unable to send email.'))
			} finally {
				this.showEMailLabel = true
				this.showEMailInput = false
				this.showEMailSending = false
			}
		},
		async copyPublicLink() {
			this.showCopyPublicLinkLabel = false
			this.showCopyPublicLinkSpinner = true

			const rootURL = generateRemoteUrl('dav')
			const token = this.calendar.publishURL.split('/').slice(-1)[0]
			const url = new URL(generateUrl('apps/calendar') + '/p/' + token, rootURL)

			// copy link for calendar to clipboard
			try {
				await navigator.clipboard.writeText(url)
				showSuccess(this.$t('calendar', 'Calendar link copied to clipboard.'))
			} catch (error) {
				console.debug(error)
				showError(this.$t('calendar', 'Calendar link could not be copied to clipboard.'))
			} finally {
				this.showCopyPublicLinkLabel = true
				this.showCopyPublicLinkSpinner = false
			}
		},
		async copySubscriptionLink() {
			this.menuOpen = true
			this.showCopySubscriptionLinkLabel = false
			this.showCopySubscriptionLinkSpinner = true
			this.showCopySubscriptionLinkSuccess = false
			this.showCopySubscriptionLinkError = false

			const rootURL = generateRemoteUrl('dav')
			const url = new URL(this.calendar.publishURL + '?export', rootURL)

			// copy link for calendar to clipboard
			try {
				await navigator.clipboard.writeText(url)
				this.menuOpen = true
				this.showCopySubscriptionLinkLabel = false
				this.showCopySubscriptionLinkSpinner = false
				this.showCopySubscriptionLinkSuccess = true
				this.showCopySubscriptionLinkError = false

				showSuccess(this.$t('calendar', 'Calendar link copied to clipboard.'))
			} catch (error) {
				console.debug(error)
				this.menuOpen = true
				this.showCopySubscriptionLinkLabel = false
				this.showCopySubscriptionLinkSpinner = false
				this.showCopySubscriptionLinkSuccess = false
				this.showCopySubscriptionLinkError = true

				showError(this.$t('calendar', 'Calendar link could not be copied to clipboard.'))
			} finally {
				setTimeout(() => {
					this.showCopySubscriptionLinkLabel = true
					this.showCopySubscriptionLinkSpinner = false
					this.showCopySubscriptionLinkSuccess = false
					this.showCopySubscriptionLinkError = false
				}, 2000)
			}
		},
		async copyEmbedCode() {
			this.menuOpen = true
			this.showCopyEmbedCodeLinkLabel = false
			this.showCopyEmbedCodeLinkSpinner = true
			this.showCopyEmbedCodeLinkSuccess = false
			this.showCopyEmbedCodeLinkError = false

			const rootURL = generateRemoteUrl('dav')
			const token = this.calendar.publishURL.split('/').slice(-1)[0]
			const url = new URL(generateUrl('apps/calendar') + '/embed/' + token, rootURL)

			const code = '<iframe width="400" height="215" src="' + url + '"></iframe>'

			// copy link for calendar to clipboard
			try {
				await this.$copyText(code)
				this.menuOpen = true
				this.showCopyEmbedCodeLinkLabel = false
				this.showCopyEmbedCodeLinkSpinner = false
				this.showCopyEmbedCodeLinkSuccess = true
				this.showCopyEmbedCodeLinkError = false

				showSuccess(this.$t('calendar', 'Embed code copied to clipboard.'))
			} catch (error) {
				console.debug(error)
				this.menuOpen = true
				this.showCopyEmbedCodeLinkLabel = false
				this.showCopyEmbedCodeLinkSpinner = false
				this.showCopyEmbedCodeLinkSuccess = false
				this.showCopyEmbedCodeLinkError = true

				showError(this.$t('calendar', 'Embed code could not be copied to clipboard.'))
			} finally {
				setTimeout(() => {
					this.showCopyEmbedCodeLinkLabel = true
					this.showCopyEmbedCodeLinkSpinner = false
					this.showCopyEmbedCodeLinkSuccess = false
					this.showCopyEmbedCodeLinkError = false
				}, 2000)
			}
		},
		async unpublishCalendar() {
			this.unpublishingCalendar = true

			const calendar = this.calendar
			try {
				await this.$store.dispatch('unpublishCalendar', { calendar })
				this.unpublishingCalendar = false
			} catch (error) {
				console.debug(error)
				this.unpublishingCalendar = false
				showError(this.$t('calendar', 'Unpublishing calendar failed'))
			}
		},

	},
}
</script>
