<!--
  - @copyright Copyright (c) 2019 Georg Ehrke <oc.list@georgehrke.com>
  -
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
	<button v-shortkey="['t']"
		class="button today"
		@shortkey="today"
		@click="today">
		{{ $t('calendar', 'Today') }}
	</button>
</template>

<script>

export default {
	name: 'AppNavigationHeaderTodayButton',
	methods: {
		today() {
			const name = this.$route.name
			const params = Object.assign({}, this.$route.params, {
				firstDay: 'now',
			})

			// Don't push new route when day didn't change
			if (this.$route.params.firstDay === 'now') {
				return
			}

			this.$router.push({ name, params })
		},
	},
}
</script>
