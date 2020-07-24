<template>
	<q-layout view="hHh lpr lff">
		<q-header elevated>
			<q-toolbar>
				<q-btn flat dense round icon="menu" aria-label="Menu" @click="leftDrawerOpen = !leftDrawerOpen" />

				<q-avatar square>
					<img src="~assets/logo.png" />
				</q-avatar>

				<q-toolbar-title>
					DoubleDice
				</q-toolbar-title>
			</q-toolbar>
		</q-header>

		<q-drawer v-model="leftDrawerOpen" show-if-above bordered content-class="bg-secondary text-grey-3">
			<q-list>
				<q-item-label header class="text-grey-1">
					Helpful Links
				</q-item-label>
				<EssentialLink v-for="link in essentialLinks" :key="link.title" v-bind="link" />
			</q-list>
		</q-drawer>

		<q-page-container class="bg-accent">
			<router-view />
		</q-page-container>
	</q-layout>
</template>

<script>
import EssentialLink from 'components/EssentialLink'

export default {
	name: 'MainLayout',

	components: {
		EssentialLink
	},

	data () {
		return {
			leftDrawerOpen: false,
			essentialLinks: [
				{
					title: 'Homepage',
					caption: 'www.archonapp.net',
					icon: 'home',
					link: 'http://archonapp.net/'
				},
				{
					title: 'Hive Account',
					caption: '@doubledice',
					icon: 'account_circle',
					link: 'https://www.peakd.com/@doubledice'
				},
				{
					title: 'Discord Channel',
					caption: 'archon / upfundme',
					icon: 'chat',
					link: 'https://discord.gg/JqnXeqG'
				},
			]
		}
	}
}
</script>
