# learn-vue
> think and talk something 

##需求：不同的用户id访问一个商家，比如收藏商家/取消收藏这个功能，让用户在登录的时候所做的操作可以保存下来，以便于下次登录的时候状态一致

``` bash
# 数据通信：App.vue中，通过初始化设置data选项中的seller.id，以及在created的时候获取数据。
data() {
  	return {
  	  seller: {
        id:(() => {
          let queryParam = urlParse();//urlParse是解析url的search数据
          return queryParam.id;
        })()
      }
    }
  },
  created() {
  	this.$http.get('/api/seller?id=' +this.seller.id).then((response) => {
  	//向服务器请求这个id的数据
  	  response = response.body;
  	  if(response.errno === ERR_OK){
  	  	this.seller = Object.assign({},this.seller,response.data);
  	  	//利用es6中的assign方法把id属性值和返回数据的属性值都设置在seller对象中
  	  }
  	});
  }

# urlParse函数
/*
解析url参数
@example ?id=12345&a=b
@return Object{id:12345,a:b}
 */
export function urlParse() {
	let url = window.location.search;
	let obj = {};
	let reg = /[?&][^?&]+=[^?&]+/g;
	let arr = url.match(reg);
	//['?id=12345','&a=b']
	if(arr){
		arr.forEach((item) =>{
			let tempArr = item.substring(1).split('=');
			let key = decodeURIComponent(tempArr[0]);
			let value = decodeURIComponent(tempArr[1]);
			obj[key] = value;
		})
	}
	return obj;
}

# 除了完成基本的url解析和请求发送后的数据拼接在seller对象中，还需要在具体的组件中利用localStorage读写和保存数据。
  在seller.vue中的配置：
data() {
	return {
		favorite: (() => {
			return loadFromLocal(this.seller.id,'favorite',false);
			//初始化读取数据
		})()
	}
}
methods:{
	toggleFavorite() {
		this.favorite = !this.favorite;
		saveToLocal(this.seller.id,'favorite',this.favorite);
		//操作收藏功能键后把状态保存
	}
}
# saveToLocal，loadFromLocal函数：

export function saveToLocal(id,key,value){
	let seller = window.localStorage.__seller__;
	if(!seller){
		seller={};
		seller[id] ={};
	}else{
		seller = JSON.parse(seller);
		if(!seller[id]){
			seller[id] ={};
		}
	}
	seller[id][key]=value;
	window.localStorage.__seller__=JSON.stringify(seller)
}
export function loadFromLocal(id,key,def){
	let seller = window.localStorage.__seller__;
	if(!seller){
		return def;
	}
	seller = JSON.parse(seller)[id];
	if(!seller){
		return def;
	}
	let ret = seller[key];
	return ret || def;
}

```

