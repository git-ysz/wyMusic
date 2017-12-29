<template>
	<el-row class="demo-autocomplete">
		<el-col :span="12">
			<div class="sub-title">输入后匹配输入歌曲</div>
			<el-autocomplete class="inline-input" v-model="songName" :fetch-suggestions="querySearch" placeholder="请输入内容" :trigger-on-focus="false" @select="handleSelect">
				<i v-if="songUrl != ''" class="el-icon-service el-input__icon el-icon-loading" slot="suffix"></i>
				<i v-else class="el-icon-service el-input__icon" slot="suffix"></i>
			</el-autocomplete>
			<el-button plain :id="songId" @click="tag(songId)">{{ state }}</el-button>
			<audio :src="songUrl" autoplay="autoplay" loop="loop"></audio>
		</el-col>
	</el-row>
</template>

<script>
	//封装Ajax
	function Ajax(type, url, data, success, failed) {
		// 创建ajax对象
		var xhr = null;
		if(window.XMLHttpRequest) {
			xhr = new XMLHttpRequest();
		} else {
			xhr = new ActiveXObject('Microsoft.XMLHTTP')
		}

		var type = type.toUpperCase();
		// 用于清除缓存
		var random = Math.random();

		if(typeof data == 'object') {
			var str = '';
			for(var key in data) {
				str += key + '=' + data[key] + '&';
			}
			data = str.replace(/&$/, '');
		}

		if(type == 'GET') {
			if(data) {
				xhr.open('GET', url + '?' + data, true);
			} else {
				xhr.open('GET', url + '?t=' + random, true);
			}
			xhr.send();

		} else if(type == 'POST') {
			xhr.open('POST', url, true);
			// 如果需要像 html 表单那样 POST 数据，请使用 setRequestHeader() 来添加 http 头。
			xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
			xhr.send(data);
		}
		// 处理返回数据
		xhr.onreadystatechange = function() {
			if(xhr.readyState == 4) {
				if(xhr.status == 200) {
					success(xhr.responseText);
				} else {
					if(failed) {
						failed(xhr.status);
					}
				}
			}
		}
	}

	export default {
		data() {
			return {
				//搜索后数据
				restaurants: [],
				state: "播放",
				songId: "",
				songUrl: "",
				songName: "",
				timeout: null
			}
		},
		methods: {
			tag(id) {
				if(id == "") {
					this.$message('请选择一首你要听的歌曲 *-*')
					return false
				}
				if(this.state == "播放") {
					let _get = "type=song&id=" + id + "&br=198000";
					Ajax( //Ajax(type, url, data, success, failed)
						'get',
						'https://api.imjad.cn/cloudmusic/',
						_get,
						(data) => {
							data = JSON.parse(data)
							//data = data.data;
							//console.log(data)
							let url = data.data[0].url
							this.songUrl = url
						},
						(error) => {
							this.$message('服务器错误，请稍后再试 *-*')
						})
					this.state = "停止"
				} else{
					this.songUrl = ""
					this.state = "播放"
				}
			},
			querySearch(queryString, cb) {
				this.song = "";
				let _song = "type=search" + "&s=" + queryString
				Ajax( //Ajax(type, url, data, success, failed)
					'get',
					'https://api.imjad.cn/cloudmusic/',
					_song,
					(data) => {
						let restaurants = this.restaurants
						data = JSON.parse(data)
						//console.log(data)
						restaurants = data.result.songs
						//console.log(restaurants)
						let endsongs = []
						for(let i = 0; i < restaurants.length; i++) {
							endsongs[i] = {
								"value": restaurants[i].name + "-" + restaurants[i].ar[0].name,
								"address": restaurants[i].id
							}
						}
						//console.log(endsongs)
						//延时加载
						clearTimeout(this.timeout);
						this.timeout = setTimeout(() => {
							cb(endsongs)
						}, 3000 * Math.random());
					},
					function(error) {
						console.log(error)
					});

			},
			handleSelect(item) {
				//选中后执行的函数
				console.log(item);
				this.songId = item.address
			}
		}
	}
</script>