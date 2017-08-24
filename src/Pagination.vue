<template>
	<div>
		<nav class="pagination" v-bind:class="this.modifiers">

			<ul v-show="totalPages > 1" class="pagination-list">

				<li>
					<a class="pagination-previous" v-bind:disabled="this.pageLinkDisabled(this.page-1)" href="javascript:void(0)" @click="prev">&lt</a>
					<a class="pagination-link" v-bind:disabled="chunkLinkDisabled(-1)" href="javascript:void(0)" @click="setChunk(-1)">&lt&lt</a>
				</li>

				<a v-for="page in pages" class="pagination-link" v-bind:class="activeClass(page)" href="javascript:void(0)" @click="setPage(page)">{{page}}</a>

				<li>
					<a class="pagination-link" v-bind:disabled="chunkLinkDisabled(1)" href="javascript:void(0)" @click="setChunk(1)">&gt&gt</a>
					<a class="pagination-next" v-bind:disabled="pageLinkDisabled(this.page+1)" href="javascript:void(0)" @click="next">&gt</a>

				</li>
			</ul>
		</nav>
		<p v-show="parseInt(this.records)" class="has-text-centered">{{this.count}}</p>
	</div>
</template>

<script>
export default {
	props: {
		for: {
			type: String,
			required: true
		},
		records: {
			type: Number,
			required: true
		},
		perPage: {
			type: Number,
			required: false,
			default: 25
		},
		chunk: {
			type: Number,
			required: false,
			default: 10
		},
		countText: {
			type: String,
			required: false,
			default: 'Showing {from} to {to} of {count} records|{count} records|One record'
		},
		modifiers: {
			type: String,
			default: ''
		}
	},
	computed: {
		page: function() {
			return this.$store.state[this.for].page
		},
		pages: function() {
			if (!this.records)
				return []

			return range(this.paginationStart, this.pagesInCurrentChunk)
		},
		totalPages: function() {
			return this.records ? Math.ceil(this.records / this.perPage) : 1
		},
		totalChunks: function() {
			return Math.ceil(this.totalPages / this.chunk)
		},
		currentChunk: function() {
			return Math.ceil(this.page / this.chunk)
		},
		paginationStart: function() {
			return ((this.currentChunk - 1) * this.chunk) + 1
		},
		pagesInCurrentChunk: function() {

			return this.paginationStart + this.chunk <= this.totalPages ?
				this.chunk :
				this.totalPages - this.paginationStart + 1

		},
		count: function() {

			let from = ((this.page - 1) * this.perPage) + 1
			let to = this.page == (this.totalPages) ? this.records : from + this.perPage - 1
			let parts = this.countText.split('|')
			let i = Math.min(this.records == 1 ? 2 : this.totalPages == 1 ? 1 : 0, parts.length - 1)

			return parts[i].replace('{count}', this.records)
				.replace('{from}', from)
				.replace('{to}', to)
		}
	},
	created: function() {
		let name = this.for

		if (this.$store.state[name]) return

		this.$store.registerModule(this.for, {
			state: {
				page: 1
			},
			mutations: {
				[`${name}/PAGINATE`](state, page) {
					state.page = page
				}
			}
		})
	},
	methods: {
		paginate: function(page) {
			console.log('paginate ' + page)
			this.$store.commit(`${this.for}/PAGINATE`, page)
		},
		setPage: function(page) {
			if (this.allowedPage(page)) {
				this.paginate(page)
			}
		},
		next: function() {
			return this.setPage(this.page + 1)
		},
		prev: function() {
			return this.setPage(this.page - 1)
		},
		nextChunk: function() {
			return this.setChunk(1)
		},
		prevChunk: function() {
			return this.setChunk(-1)
		},
		setChunk: function(direction) {
			this.setPage((((this.currentChunk - 1) + direction) * this.chunk) + 1)
		},
		allowedPage: function(page) {
			return page >= 1 && page <= this.totalPages
		},
		allowedChunk: function(direction) {
			return (direction == 1 && this.currentChunk < this.totalChunks) ||
				(direction == -1 && this.currentChunk > 1)
		},
		pageLinkDisabled: function(direction) {
			return this.allowedPage(direction) ? false : true
		},
		chunkLinkDisabled: function(direction) {
			return this.allowedChunk(direction) ? false : true
		},
		activeClass: function(page) {
			return this.page == page ? 'is-current' : ''
		}
	}
}

function range(start, count) {
	return Array.apply(0, Array(count))
		.map(function(element, index) {
			return index + start
		})
}

</script>

<style lang="css">

</style>
