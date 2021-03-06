<template>
	<b-container fluid>
		<h1 class="mb-5">{{$t('content.category.title')}}</h1>
		<delete-modal
			:model-title="$t('content.category.deleteModal.title')"
			:message="$t('content.category.deleteModal.msg')"
			@ok="deleteCategory(deleteId)"
		></delete-modal>
		<b-row>
			<b-col cols="8">
				<b-card :title="$t('content.category.baseInfo')">
					<b-row>
						<b-col>
							<b-form-group
								:label="$t('content.category.name')"
								label-for="category-name"
							>
								<b-form-input 
									id="category-name"
									v-model="category.name"
									:placeholder="$t('content.category.name')"
								></b-form-input>
							</b-form-group>
						</b-col>
						<b-col>
							<b-form-group
								:label="$t('content.category.parent')"
								label-for="category-parent"
							>
								<b-form-select
									id="category-parent"
									v-model="category.parent" 
									:options="renderSelectCategory"
								></b-form-select>
							</b-form-group>
						</b-col>
					</b-row>
					<b-row>
						<b-col>
							<b-form-group
								:label="$t('content.category.comment')"
								label-for="category-comment"
							>
								<b-form-textarea 
									id="category-comment"
									v-model="category.comment"
									rows="3"
									:placeholder="$t('content.category.commentPlaceholder')"
								></b-form-textarea>
							</b-form-group>
						</b-col>
					</b-row>
					<b-row>
						<b-col class="ml-auto" cols="auto">
							<b-btn v-if="isCreate" @click="reset">{{$t('content.category.reset')}}</b-btn>
							<b-btn v-if="isCreate" variant="primary" @click="onSubmit">{{$t('content.category.create')}}</b-btn>
							<b-btn v-if="!isCreate" @click="back">{{$t('content.category.back')}}</b-btn>
							<b-btn v-if="!isCreate" variant="primary" @click="updateCategoryById()">{{$t('content.category.update')}}</b-btn>
						</b-col>
					</b-row>
				</b-card>
			</b-col>
			<b-col cols="4">
				<b-card :title="$t('content.category.list')" style="height: 338px"> 
					<b-pagination
							v-model="curPage"
							small
							limit="3"
							:per-page="perPage"
							:total-rows="rows"
							align="center"
					></b-pagination>
					<b-table
						id="category"
						hover
						small
						:current-page="curPage"
						:per-page="perPage"
						:fields="[
							{ key: 'name', label: $t('content.category.name')},
							{ key: 'parent', label: $t('content.category.parent')},
							{ key: 'action', label: $t('content.category.action')}
						]"
						:items="categoryList"
					>
						<template slot="name" slot-scope="data">
							<b-btn
								variant="link"
								@click="getCategoryById(data.item.id)"
							>{{data.item.name}}</b-btn>
						</template>
						<template slot="parent" slot-scope="data">
							{{ data.item.parent === null ? $t('content.category.default') : indexOf(data.item.parent)}}
						</template>
						<template slot="action" slot-scope="data">
							<i 
								v-b-modal.delete-item
								class="fa fa-trash fa-lg text-danger"
								aria-hidden="true"
								@click="getCategoryId(data.item.id)"
							></i>
						</template>
					</b-table>
				</b-card>
			</b-col>
		</b-row>
		<b-card class="my-4" style="height: 430px; overflow: auto">
			<vue2-org-tree
				:data="categoryTree"
				horizontal
				:render-content="renderContent"
			></vue2-org-tree>
		</b-card>
	</b-container>
</template>

<script>
import Vue2OrgTree from 'vue2-org-tree';
import DeleteModal from '../utils/DeleteModal.vue';

export default {
	components: { Vue2OrgTree, DeleteModal },
	data() {
		return {
			curPage: 1,
			perPage: 4,
			isCreate: true,
			category: {
				id: '',
				name: '',
				parent: null,
				comment: ''
			},
			deleteId: '',
			categoryList: [],
			categoryOptions: [],
		};
	},
	computed: {
		rows() {
			return this.categoryList.length;
		},
		renderSelectCategory() {
			return this.categoryOptions.filter(category => {
				return category.value !== this.category.id;
			});
		},
		categoryTree() {
			return {
				id: 0, label: 'root', children: this.contructTree()
			};
		}
	},
	mounted() {
		this.getCategoryOption();
		this.getCategoryList();
	},
	methods: {
		contructTree() {
			const tree = {};
			const result = [];

			this.categoryList.forEach(category => {
				tree[category.id] = {
					id: category.id,
					label: category.name,
					children: [],
					parent: category.parent
				}
			});

			for (let key in tree) {
				const { parent } = tree[key];

				if (parent) {
					tree[parent].children.push(tree[key]);
				}
			}

			for (let key in tree) {
				const { parent } = tree[key];

				if (!parent) {
					result.push(tree[key]);
				}
			}

			return result;
		},
		renderContent(h, data) {
			return data.label;
		},
		getCategoryList() {
			return this.$api.category.getList().then(res => {
				this.categoryList = res.data;
			});
		},
		getCategoryOption() {
			this.categoryOptions = [ {value: null, text: '请选择父类'} ];
			return this.$api.category.getList().then(res => {
				res.data.forEach(item => {
					const option = { value: item.id, text: item.name };
					this.categoryOptions.push(option);
				});
			});
		},
		onSubmit() {
			this.$api.category.create(this.category).then(() => {
				this.getCategoryList();
			}).then(() => {
				this.getCategoryOption();
				this.reset();
			});
		},
		reset() {
			this.category.id = '';
			this.category.name = '';
			this.category.comment = '';
			this.category.parent = null;
		},
		deleteCategory(id) {
			return this.$api.category.delete(id).then(() => {
				this.getCategoryList();
				this.getCategoryOption();
			}).catch(e => {
				console.log(e);
			});
		},
		getCategoryById(id) {
			return this.$api.category.get(id).then(res => {
				this.category.name = res.data.name;
				this.category.comment = res.data.comment;
				this.category.parent = res.data.parent;
				this.category.id = res.data.id;
			}).then(() => {
				this.isCreate = false;
			});
		},
		getCategoryId(id) {
			this.deleteId = id;
		},
		updateCategoryById() {
			return this.$api.category.update(this.category).then(() => {
				this.getCategoryList();
				this.getCategoryOption();
				this.reset();
			}).then(()=> {
				this.isCreate = true;
			}).catch(e => {
				console.log(e);
			});
		},
		back() {
			this.isCreate = true;
			this.reset();
		},
		indexOf(val) {
			let result;
			this.categoryOptions.forEach(item => {
				if(item.value === val) {
					result = item.text;
				}
			});
			return result;
		}
	}
};
</script>