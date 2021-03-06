<template>
<div class="entry" :class="{folder : isFolder, node: !isFolder}">
	<div v-if="!editing&&!adding"
	class="nodeTop"
	@mouseover="hover = true"
	@mouseleave="hover = false">
		<input type="checkbox" v-model="checked" @change="checkMark">
		<h4 @click="nodeClick" :title="name">{{ name }}</h4>
		<div class="nodeTopRight">
			<span @click="addNode" v-show="canAdd"
			title="Add to Folder">Add</span>
			<span @click="openEdit" v-show="canEdit" title="Edit">Edit</span>
			<span @click="removeNode" v-show="canEdit" title="Delete">Delete</span>
			<span @click="nodeClick" title="Open Folder"  v-if="isFolder">{{isOpen? "[-]" : "[+]"}}</span>
		</div>
	</div>
	<div v-if="editing||adding" class="nodeEdit">
		<h4>{{adding?"Add":"Edit"}} {{getType}}</h4>
		<div class="typeToggle" @click="typeToggle">
		Current type: {{isFolder? "folder" : "node"}}.
		</div>
		<br>
		<div :class="{title : isFolder, nodeRight: !isFolder}">
			<div :title="name">
			<label>Enter a name:</label> &nbsp; <input v-model="name" />
			</div>
		</div>
		<div class="editButtons">
			<button @click="saveNode">Save</button>
			<button @click="cancelEdit">Cancel</button>
		</div>
		<br>
	</div>
	<div v-if="isFolder" class="nodes" v-show="isOpen">
		<div v-if="noChildren"><br>No nodes or folders here.</div>
		<Node @child-clicked="childClicked"
		@cancel-add="editChild=false"
		@edit-child="editChild=true"
		@remove-child="removeChild"
		@save-child="saveChild"
		v-for="node in children"
		:key="node.id"
		:node="node"
		:checkState="checkAll" ></Node>
	</div>
</div>
</template>

<script>
export default {
	name: 'Node',
	props: {
		node: Object,
		checkState:Number
	},
	data() {
		return {
			'id': this.node.id,
			'name': this.node.name,
			'children': this.node.children,
			'isFolder': this.node.isFolder,
			'isOpen' : this.node.parent_id==null,
			'checked' : false,
			'checkAll' : 0,
			'childrenChecked' : 0,
			'editing' : false,
			'adding' : this.node.adding,
			'editChild' : false,
			'hover':false,
		}
	},
	watch: {
		checkState() {
			if(this.checkState!=0)
				this.checkAll = this.checkState;
			if(this.checkState == 1)
				this.checked = true;
			if(this.checkState == -1)
				this.checked = false;
		},
		checked(){
			this.$emit('child-clicked', {c:this.checked,r:false})
		}
	},
	computed: {
		hasChildren(){
			return this.children!=null&&this.children.length>0;
		},
		canEdit(){
			return !this.editChild && this.hover;
		},
		canAdd(){
			return !this.editChild && this.isFolder && this.isOpen &&this.hover
		},
		getType(){
			return this.isFolder?"Folder" : "node"
		},
		noChildren(){
			return this.isOpen&&!this.hasChildren;
		},
		nChildren(){
			return this.children==null? 0 : this.children.length;
		}
	},
	methods: {
		childClicked(d) {
			if(d.r) {
				this.checkAll = 0;
				return;
			}
			let n = this.childrenChecked;
			d.c ? n++ : n--;
			n = n>0 ? n : 0;
			this.childrenChecked = n>this.nChildren ? this.nChildren : n;
			if(this.childrenChecked < this.nChildren){
				this.checked = true;
			}else{
				this.checked = false;
			}
			if(this.childrenChecked == 0){
				this.checked = false;
			}else if(this.childrenChecked == this.nChildren){
				this.checked = true;
			}
		},
		nodeClick() {
			if(this.isFolder&&!this.editChild) {
				this.isOpen = !this.isOpen;
			}else if(!this.isFolder){
				this.checked = !this.checked;
				this.checkMark()
			}
		},
		typeToggle(){
			if(this.adding)
				this.isFolder = !this.isFolder;
		},
		checkMark(){
			this.$emit('child-clicked', {c:this.checked,r:true})
			if(this.hasChildren){
				if(this.checked) {
					this.checkAll = 1;
					this.childrenChecked = this.nChildren
				}else{
					this.checkAll = -1;
					this.childrenChecked = 0
				}
			}
		},
		openEdit(){
			this.editing = true;
			this.isOpen = false;
			this.$emit('edit-child');
		},
		addNode() {
			this.editChild = true;
			let n =  {
				id: "node_"+ Math.random().toString(36).substr(2, 9),
				name: "",
				parent_id:this.id,
				order: -1,
				adding: true,
				children:null
			};
			if(this.children==null){
				this.children = [];
			}
			this.children.push(n);

			this.children.sort(
				(a, b) => {
					if (a.order > b.order)
						return 1;
					else if (a.order < b.order)
						return -1;
					return 0;
				})
		},
		saveNode() {
		this.editing=false;
		this.adding=false;
		this.$emit('save-child',this);
		},
		removeNode(){
			this.$emit('remove-child',this.id);
		},
		cancelEdit(){
			if(this.node.adding){
				this.$emit('remove-child',this.node.id);
			}else{
				this.editing=false;
			}
			this.$emit('cancel-add');
		},
		removeChild(id){
			const index = this.children.findIndex(b => b.id == id)
			if(index>=0)
				this.children.splice(index, 1)
		},
		saveChild(node){
			const index = this.children.findIndex(item => item.id == node.id)
			const found = this.children[index];
			for(let key in found){
				this.children[index][key] = node[key];
			}
			this.editChild = false;
		}
	}
}
</script>
<style>
body {
	font-family: Menlo, Consolas, monospace;
	color: #444;
}
.container{
width:50%;
	margin:auto;
}
.nodeTop h4{
	margin:0px;
	margin-right:5em;
	display:inline-block;
}
.nodeTopRight, .editButtons{
	float:right;
}
.nodeTopRight span{
	padding-left:0.5em;
}
.nodeTopRight span, .toggleType,.nodeTop h4,.typeToggle{
	cursor:pointer;
}
.entry{
	margin: 0.5em 0em;
	padding: 0.25em 0em 0.25em 0em;
}
.nodeTop:hover{
	background-color: #f1f1f1;
}
.folder .nodes {
	margin-left: 1em;
}
</style>