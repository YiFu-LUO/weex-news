<template>
	<div class="content">
		<!-- 
			weex组件文档
			https://weex.apache.org/zh/docs/components/slider.html
		--> 
		
		<!-- 标题栏 -->
		<div class="page-header">
			<!-- 状态栏占位 -->
			<div :style="{ height: statusBarHeight }"></div>
			<div class="page-header-wrapper">
				<div class="page-header-left">
					<!-- <text class="logo">Logo</text> -->
					<text class="logo yticon">&#xe615;</text>
				</div>
				<div class="page-header-center">
					<text class="search-input">我的订单</text>
				</div>
				<div class="page-header-right">
					<image class="contribute-icon" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAAC20lEQVRoQ+1Y7VEVQRDsjkAzUCNQIgAjUCNQIhAjECNQIhAjQCIQMsAIxAwggraa2qXO43bvdu+LenVbxQ/q7e5Nz/TM9CyxI4s7ggMbkMcWyS0iW0Rm8sBGrZkcW33tqIhIeg7gI4BXHRZcATgheV1tXcHBaiCSbPwvAE8z37sB8JqkQc26qoBIsvF/AohbAF2GGugTAI7IHkmDmm0VAwkgHIlIp3ckf7YtDBG7CGCuSO7NhgIolyiSvgP4EIz6RPJbykBJ3uf9XqckD+cCk42IJCfyEQAndXv9IBkBJe2TdAzg80gApq4dcZK6JwlE0lsAZ4mD5yT9+6Al6RTA+0Gb85s6aewjOSDm/RsAvwE0c+CCpLlftELOHPRUudSdPrcPIOnAHBAb68NfSJoeq60GPS9JGtSDtQFZMjxTRcQNrS0z/L9Lb7LJhZxwjsV+48pjjhd3+amApJx/SNLV6L8V9Jd7RyeXATj3fHawBhsLxJ50kndpKRtx1I5Ih/5qypcoWQy8SIONAlKTA5JMm5cADOC43fUlubl+DXcPli2LAmnJkWTjajXaTnp20DWqg/LyWxqRRjSSH4t3Soo9qnevzywdEQVDs0IyGHZPMZK9Cnw0EEn7JC+HREdSBOJBKithJLmieRTA7EAaQm+QRJHkSuRBqnd/w8O3JHMT5p0PR0WkgsdRZF6TfJGLoiSX72cASkeB8mSvAHJPl9wQ1ZL0vTRcPCLhg825w3nixL+TJKFZuofEjj8oGmsBMd8Nxhort849Lg99kFg0R5pWh+boJuY8aK6/oeM/0Gg9OVXfEEtzpMuQQKdYlW5qlO8q1BrSb2r2rEatGmMfNbWmAjQ2IrGUumMXT3VTgQj3+F3Nf8mSnXt88CAUnzwntqvqOs84B6mC0ffSaC/4NTE1tlZZVHHIDvVLY3I87pXQFR9d5cgGZBW3Zz66RWSLyEwe2Kg1k2Orr92ZiPwDeAycQswzKBAAAAAASUVORK5CYII=" />
					<text class="contribute-text">投稿</text>
				</div>
			</div>
		</div>
		
		
		<!-- uni 官方顶部选项卡组件 -->
		<uni-tab-bar :drag="true" :tab-bars="tabBars" :tab-index="tabCurrentIndex" @tabChange="tabChange" class="uniTabbar"></uni-tab-bar>
		
		<!-- slider就是uni swiper -->
		<slider class="slider" :index="tabCurrentIndex" :infinite="false" @change="tabChange">
			
			<!-- list 垂直滚动列表组件 -->
			<list v-for="(tabItem, tabIndex) in tabBars" :key="tabIndex" class="list-content">
				<!-- refresh 下拉刷新组件  
				* 	 weex 的refresh和loading组件在ios效果很好，但是在安卓端效果并不好
				* -->
				<refresh class="loading" @refresh="onRefresh" :display="tabItem.refreshing ? 'show' : 'hide'">
					<loading-indicator v-if="tabItem.refreshing" class="loading-icon"></loading-indicator>
					<text class="loading-text">{{tabItem.refreshing?'正在加载..': '下拉刷新数据'}}</text>
				</refresh>
				
				<!-- 新闻列表 -->
				<cell v-for="(item, index) in tabItem.newsList" :key="index" class="news-item" @click="navToDetails(item)">
					<text :class="['title', 'title'+item.type]">{{item.title}}</text>
					<div v-if="item.images.length > 0" :class="['img-list', 'img-list'+item.type, item.images.length === 1 && item.type===3 ? 'img-list-single': '']">
						<div 
							v-for="(imgItem, imgIndex) in item.images" :key="imgIndex"
							:class="['img-wrapper', 'img-wrapper'+item.type, item.images.length === 1 && item.type===3 ? 'img-wrapper-single': '']"
						>
							<image class="img" :src="imgItem" />
							<view class="video-tip">
								<image class="video-tip-icon" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAAEC0lEQVRoQ+2ajVEVMRDHdzuwA6ACpQKxArECtQKxAqECoQKhAqECoQKxAqEDrWCdn7Nx8vJy+bp3T4YhM2+O8S7J/rO7//2IKo9k6CPBIU9Acpo0s10ReSkiPA8mtH0tIncicqOqPDcyZmvEhX8rIu8cQI9gtyJyKSIXc0ENA3EAnxxAj/BT356LyEdV/TWyWDcQM3smIgA4mtjwXkQ4aX4Mngj3QkSYy5PfTmb+laoeLg7EzBDga8aEEB4TOVfVAKAoj2sUc+QXQC0PxMzY8Esi3W8ROVbV05FTDHPMDC1AEBzEcqY1AeLMQQxtXANuZvjMa/cb/i6Oqo9kQKCFI1WtLl7bfOq9mUHd3/w9ND1F5f+WKAJxn/gebQiIg1Y/mAEEUsDX8J0zVZ0iljoQZydAYLuMrYCIwXOQrYc2qREzw4E/RAu/X9KcRrUX5mWBODX+jBY/UdXjuZuNznd5PnscepNjtikgODJpRzCp3VFaHBU+MTEOkSDMIJ0hFKyMNSAZbZA2NMUJn7ujqjebABDWyDDnXpqb5YDEvnGvqsHZi7I5CMgBxiHDxRx5bmSYGZlyyADWmCwHBN8IwjdRH5Im3B+En5UIJuYFBeMnjFtV3Y/frwDJmNV+K/1NAGEvIv+pqp7MUU1GthXzSoHE+VSzWRU0EsuOaUDhw+aWmNdKOEiBxOzQlYkWNJIqAiAI0V0dmhkZNvkXYyUkpEDYhFJ17cOaWXQACUtxaPhgc9JpZvFBr+Rg/xNI8B+0w0lXR0LDzUCIoE0bNPpISdC1uJD7uJQVlzTyEIFQgFGhMpo10pVfDfgIwlAiU9s0af4h+gglARkE8WURZ98G/V65Fhal3zgg3qnqXpVK/IMG0/rhAOYExDh9KgZEcqy4DtlEirKpTgutqLjsnk5RnEaLWeaUhiY0srFOS1KxrqVPtTS+2by8xsdsnkONNN5G0pDCQcVmtcaoLYVVV63e0zDo8L+0OVgvrNy84lIXemRiM022CtjynWsabVCwMdpKXQeSOlZXcGwRsPWbJAgyLZvOPOh2UKZWn6xYS0Dibl/IVF+1VoytJ15wbqyCtmkwKdIZGnZZE+9tmbLI4mC8VRuDAG8xpo00sQFDi2iRJrabU2jGBYVVmbMKxJ0/dzfSXeGVzM3ZiRZt2tGsgmDdJiAFMGiHNPxijk+YGV1NsuHgD82aCB82A4lomdohvf8jrQm3s61XbzgzAMJtVXwWOPZhD7F0AXEwnBrqjzv1sRCACnfp/HvIdsNlTbiDn+pgDuVn3UCCxN4wA1Bods+xrr8R26/yuuuULh8p8D0nSzsTE8ldOZcAhttgKsUhAEM+Ujty1xIm1PJfOK7nCh/LM2xaNVDbfv8EZNsnXtvvDyrmF1FIBKIwAAAAAElFTkSuQmCC" />
							</view>
						</div>
					</div>
					<div v-else class="img-empty"></div>
					<div :class="['bot', 'bot'+item.type]">
						<text class="author">{{item.author}}</text>
						<text class="time">{{item.time}}</text>
					</div>
				</cell>
				
				<!-- 加载更多组件 -->
				<loading class="loading" @loading="loadMore(tabItem)" :display="tabItem.loadMoreStatus==1 ? 'show' : 'hide'">
					<loading-indicator v-if="tabItem.loadMoreStatus==1" class="loading-icon"></loading-indicator>
					<text class="loading-text">{{tabItem.loadMoreStatus==0?'上拉显示更多':tabItem.loadMoreStatus==1?'正在加载..':'没有更多数据了'}}</text>
				</loading>
			</list>
		 
		</slider>
	</div>
</template>

<script>
	import indexMixin from './common/index'
	import uniTabBar from './components/tabbar.vue'
	const domModule = weex.requireModule('dom')

	export default {
		/**
		 * 大部分js可以复用vue中写的
		 * 直接混入即可
		 */
		mixins: [indexMixin],  
		components: {
			uniTabBar,
		},
		data(){
			return {
				statusBarHeight: '0wx', //状态栏占位高度
			}
		},
		beforeCreate(){
			const domModule = weex.requireModule('dom')
			domModule.addRule('fontFace', {
			    'fontFamily': "yticon",
			    'src': "url('https://at.alicdn.com/t/font_1078604_3mrhac2o3oi.ttf')",
			});
		},
		created() {
			//获取状态栏高度给顶部占位节点
			// uni.getSystemInfo({
			// 	success: res=>{
			// 		this.statusBarHeight = res.statusBarHeight + 'wx';
			// 	}
			// })
			//获取数据，方法通过mixin混入
			this.loadTabbars();
		},
		methods: {
	
			tabChange(e) {
				this.tabCurrentIndex = e.index;
				
				//第一次切换tab，动画结束后需要加载数据
				let tabItem = this.tabBars[this.tabCurrentIndex];
				if(this.tabCurrentIndex !== 0 && tabItem.loaded !== true){
					this.loadNewsList('add');
					tabItem.loaded = true;
				}
			},

			//下拉刷新
			onRefresh(e){
				this.loadNewsList('refresh');
			},
			//加载更多
			loadMore(tabItem){
				this.loadNewsList('add');
			}
		}
	}
</script>

<style>
	/* 字体图标 */
	.yticon {
		font-family: yticon;
	}
	/**
	 * weex css限制
	 * 选择器不支持嵌套
	 * 子节点不继承父节点样式（重要）
	 * 仅支持 flex布局 （这个还不错）， 默认为display:flex; flex-direction:column;
	 * 
	 * 注：我对weex也是一知半解，有说错的麻烦指出
	 */

	.content{
		flex: 1;
		background-color: #f6f6f6;
	}
	/* 顶部标题栏 */
	.page-header{
		background-color: #ffffff;
    margin-bottom: 16px;
	}
	.page-header-wrapper{
		flex-direction: row;
		justify-content: center;
		align-items: center;
		height: 100px;
		padding: 0px 20px;
	}
	.page-header-left{
		opacity: 0.9;
	}
	.logo{
		font-size: 40px;
		color: #fff;
	}
	.page-header-center{
		flex: 1;
		padding: 0px 30px 0 20px;
	}
	.search-input{
		height: 60px;
		font-size:28px;
		color: #333333;
		text-align: center;
		line-height: 60px;
	}
	.page-header-right{
		width: 50px;
		align-items: center;
	}
	.contribute-icon{
		width: 50px;
		height: 44px;
	}
	.contribute-text{
		font-size: 20px;
		color: #fff;
	}
	
	.uniTabbar{
    margin-bottom: 16px;
  }

	.slider{
		flex: 1;
		background-color: #f6f6f6;
    padding: 0px 32px;
	}
  .slider>ul{
        flex: 1;
  }
	.list-content{
		flex: 1;
    background-color: transparent;
    width: 100%;
	}
	/* 下拉刷新 加载更多 */
	.loading {
		flex-direction: row;
		align-items: center;
		justify-content: center;
		width: 750px;
		height: 80px;
	}
	.loading-text{
		font-size: 28px;
		color: #888;
	}
	.loading-icon{
		height: 40px;
		width: 40px;
		color: #999999;
		margin-right: 10px;
	}
	
	
	

	/* 新闻列表  */
	.news-item{
		width: 100%;
    padding: 0.32rem 0.4rem;
    border-bottom-width: 1px;
    border-color: #eee;
    background-color: #fff;
    margin-bottom: 10px;
    border-radius: 8px;
	}
	.title{
		font-size: 32px;
		color: #303133;
		line-height: 46px;
	}
	.bot{
		flex-direction: row;
	}
	.author{
		font-size: 26px;
		color: #aaa;
	}
	.time{
		font-size: 26px;
		color: #aaa;
		margin-left: 20px;
	}
	.img-list{
		flex-shrink: 0;
		flex-direction: row;
		background-color: #fff;
		width: 220px;
		height: 140px;
	}
 	.img-wrapper{
		flex: 1;
		flex-direction: row;
		height: 140px;
		position: relative;
	}
	.img{
		flex: 1;
	}
	.img-empty{
		height: 20px;
	}
	.video-tip{
		position: absolute;
		left: 0;
		top: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		flex: 1;
		background-color: rgba(0,0,0,.3);
	}
	/* 图在左 */
	.img-list1{
		position:absolute;
		left: 30px;
		top: 24px;
	}
	.title1{
		padding-left: 240px; 
	}
	.bot1{
		padding-left: 240px; 
		margin-top: 20px;
	}
	/* 图在右 */
	.img-list2{
		position:absolute;
		right: 30px;
		top: 24px;
	}
	.title2{
		padding-right: 210px; 
	}
	.bot2{
		margin-top: 20px;
	}
	/* 底部3图 */
	.img-list3{
		width: 700px;
		margin: 16px 0px;
	}
	.img-wrapper3{
		margin-right: 4px;
	}
	/* 底部大图 */
	.img-list-single{
		width: 690px;
		height: 240px;
		margin: 16px 0px;
	}
	.img-wrapper-single{
		height: 240px;
		margin-right: 0px;
	}
</style>
