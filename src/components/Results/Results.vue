<script setup lang="ts">
import RepoInfo from './RepoInfo.vue';
import RepoDetails from './RepoDetails.vue';
	const props = defineProps({
		searchQuery: String,
		sortElement: String,
		sortOrder: String
	})
</script>

<template>
	<div class="results">
		<div v-if="!showRepoDetails">
			<div v-if="repositories.length">
				<div v-for="(result, index) in repositories.slice(page -1, page + 3)" :key="index" >
					<RepoInfo class="results__repo-info" @click="handleRepoClick(result)" :repository="result"></RepoInfo>
					<hr>
				</div>
				<v-pagination
					v-model="page"
					:length="5"
					rounded="circle"
				></v-pagination>
			</div>
			<div v-else class="results__empty-search">
				<p>Type a keyword of the repository you want to find in the above search bar</p>
			</div>
		</div>
		<div v-else>
			<RepoDetails @close="hideDetails()" :repository="currentRepo"></RepoDetails>
		</div>
	</div>
</template>

<style lang="scss" scoped>
@import '../../assets/index.scss';
	.results {
		width: 100%;
		border-radius: 50px;
		background-color: $background-color-secondary;
		padding: 10px 30px;

		&__empty-search {
			margin: 100px 0;
			text-align: center;
		}

		&__repo-info {
			cursor: pointer;

			&:hover {
				color: $text-color-links;
			}
		}
	}
</style>

<script lang="ts">
	export default {
		name: 'explorerView',
		data() {
			return {
				repositories: [],
				page: 1,
				showRepoDetails: false,
				currentRepo: {}
			}
		},   
		methods: {
			async getSearchResult() {
				try {
					if (this.searchQuery) {
						const endpoint = "https://api.github.com/search/repositories"
						let sortQueries = ''
						if (this.sortOrder || this.sortElement) {
							sortQueries = '&sort=' + this.sortElement + '&order=' + this.sortOrder
						}
						const endpointWithQueries = endpoint + "?q=" + this.searchQuery + sortQueries
						const response = await fetch(endpointWithQueries)
						if (response.status === 404) {
							throw new Error('Page not found')
						} else if (response.status === 500) {
							throw new Error('Server error')
						} else if (!response.ok) {
							throw new Error(`HTTP error. Status: ${response.status}`)
						}
						const responseJSON = await response.json();
						this.repositories = responseJSON.items
					}
				} catch (error) {
					console.error(error);
				}
			},

			handleRepoClick(repo) {
				this.showRepoDetails = true
				this.currentRepo = repo
				this.$emit('showingDetails', true)
			},

			hideDetails() {
				this.showRepoDetails = false
				this.$emit('showingDetails', false)
			}
		},

		watch: {
			searchQuery: function () {
				this.getSearchResult()
			},
			sortOrder: function() {
				this.getSearchResult()
			},
			sortElement: function() {
				this.getSearchResult()
			}
		},

		created() {
			this.getSearchResult();
		}
	}
</script>