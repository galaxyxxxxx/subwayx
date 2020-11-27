<template>
	<el-menu :default-active="activeIndex" class="el-menu-demo" mode="horizontal" @select="handleSelect" text-color="#45617e" active-text-color="#5595a8">
		<el-menu-item class="el-menu-text-first" index="0" disabled style="font-weight: bold; color: #45617e; opacity: unset; letter-spacing: 2px;">DATA STRUCTURE PROJECT</el-menu-item>
		<el-menu-item  index="1">HOME</el-menu-item>
		<el-submenu  index="2">
			<template slot="title">About</template>
			<el-menu-item index="2-1">Introduction</el-menu-item>
			<el-menu-item index="2-2">License Agreement</el-menu-item>
			<el-menu-item index="2-3">Github</el-menu-item>
		</el-submenu>
		<el-menu-item  index="3"><a href="http://60.205.155.131/" style="text-decoration: none;">Contact</a></el-menu-item>
	</el-menu>
</template>

<script>
	export default {
		data() {
			return {
				activeIndex: '1',
				activeIndex2: '1'
			};
		},
		methods: {
			handleSelect(key, keyPath) {
				console.log(key, keyPath)
			}
		}
	}
</script>

<style>
	
	.el-menu-text-first {
		/* left: 62% !important; */
		width: 65%;
		text-align: center;
		
	}
	
	el-menu-item {
		margin-left: 30px;
	}
</style>
