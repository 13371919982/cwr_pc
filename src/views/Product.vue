<template>
  <div class="product">
    <div class="bread">
      <span>首页 >> {{ $route.params.kind }}</span>
    </div>
    <div class="container">
      <div class="left">
        <div class="price">
          <h3>价格</h3>
          <div class="price-box">
            ￥<input type="text" placeholder="最低价" v-model="minPrice"> - <input type="text" placeholder="最高价" v-model="maxPrice"><button @click="priceBtn">确定</button>
          </div>
        </div>
        <div class="content" v-for="(item,index) of list" :key="index">
          <div class="top">
            <h3>{{ item.title }}</h3>
            <span @click="btnHandler(item,index)">{{ item.sub }}</span>
          </div>
          <div class="list" :class="{active:item.isAcitve}">
            <div class="list-box">
              <div v-for="(title,index) of item.titles" :key="index" >
                <router-link :to="`/product/${item.url}/${title}`">
                  <span class="look"></span>
                  <span class="title">{{ title }}</span>
                </router-link>
              </div>
              <!-- <div class="scorll-bar"></div>
              <div class="scorll"></div> -->
            </div>
          </div>
        </div>
      </div>
      <div class="right">
        <div class="right-top">
          <div class="list" v-for="(item,index) of productList.slice((currentPage-1)*pageSize,currentPage*pageSize)" :key="index">
            <router-link :to="`/detail/${item.lid}`">
              <img :src="item.img" alt="">
            </router-link>
            <p>{{ item.brand }}</p>
            <p>{{ item.detail }}</p>
            <hr>
            <p>{{ item.price | money }}</p>
            <div class="spec">
              <p>{{ item.spec_one }}</p>
              <p>{{ item.spec_two }}</p>
              <p>{{ item.spec_three }}</p>
              <p>{{ item.spec_four }}</p>
            </div>
          </div>
        </div>
        <h1 v-if="productList.length<=0">没有搜索到您要的商品！</h1>
        <!-- total  总页数
             sizes  每页大小
             prev   上一页
             pager  当前页
             next   下一页
             jumper 跳转到那一页
             :page-sizes="pageSizes"
         -->
        <el-pagination v-if="productList.length>0"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :page-size="pageSize"
          layout="total, prev, pager, next, jumper"
          :total="productList.length">
        </el-pagination>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  data(){
    return{
      productList:[],
      minPrice:'',
      maxPrice:'',
      list:[
        { id:1, title: '排序', sub:'∨', isAcitve:false, titles:[ '默认排序', '从低到高', '从高到低'], url: 'rank'},
        { id:2, title: '性别', sub:'∨', isAcitve:false, titles:[ '男', '女'], url: 'sex'},
        { id:3, title: '尺码', sub:'∨', isAcitve:false, titles:[ '5岁', '6岁', '7岁', '8岁'], url: 'size'},
        { id:4, title: '品牌', sub:'∨', isAcitve:false, titles:[ 'AMELIE WANG', 'GIVENCHY ACCESSORIES', 'BILLIEBLUSH', 'CHLOE', 'MANTIS 麦恩缇斯', 'KARL LAGERFELD KIDS', 'MINI RODINI', 'APTAMIL'], url: 'brand'},
        { id:5, title: '颜色', sub:'∨', isAcitve:false, titles:[ '黑色', '白色', '黄色', '粉红色', '浅绿色'], url: 'color'},
      ],
      // 当前页数
      currentPage:1,
      // 每页显示个数选择器
      // pageSizes:[3, 6, 12, 24],
      // 每页大小
      pageSize:6,
    }
  },
  methods:{
    // 1.手风琴
    btnHandler(item,index){
      this.list.forEach(elem => {
        elem.sub='∨';
        if(elem.isAcitve!==this.list[index].isAcitve){
          elem.isAcitve=false;
        }
      });
      item.isAcitve=!item.isAcitve;
      if(item.isAcitve){
        item.sub='∧'
      }
    },

    // 2. price 价格查询
    priceBtn(){
      this.productList='';
      this.minPrice=parseInt(this.minPrice);
      this.maxPrice=parseInt(this.maxPrice);
      if(this.minPrice>this.maxPrice){
        let num='';
        num=this.maxPrice;
        this.maxPrice=this.minPrice;
        this.minPrice=num;
      }
      this.minPrice?this.minPrice:this.minPrice='';
      this.maxPrice?this.maxPrice:this.maxPrice='';
      this.axios.get(`/product/price`,{params:{
        minPrice:this.minPrice,
        maxPrice:this.maxPrice
      }}).then(res=>{
          this.productList=res.data;
      })
    },
  
    // 8.分页
    handleSizeChange(currentPage){
      // 改变每页显示的条数
      this.PageSize=currentPage;
      // 注意：在改变每页显示的条数时，要将页码显示到第一页
      this.currentPage=1;
    },

    handleCurrentChange(currentPage){
      // 改变默认的页数
      this.currentPage=currentPage;
    },

    // 8.keyWords 关键字查询
    kwords(kwords){
      this.axios.get('/product/kwords',{params:{
        kwords
      }}).then(res=>{
        if(res.data!=1)
          this.productList=res.data;
        else
          this.sendMsg='没有搜索到您要的商品！';
      })
    },

    // Header组件 传过来的参数kind
    // 9.kind 分类
    head(kind){
      this.axios.get(`/product/list`,{params:{
        kind
      }}).then(res=>{
        this.productList=res.data;
      })
    }
  },
  created(){
    let url=this.$route.path.split('/');
    if(url[2]==='head'){
      this.head(url[3])
    }else{
      this.kwords(url[3]);
    }
  },
  watch:{
    '$route'(to){
      // 路由跳转监听
      let url=to.path.split('/');
      if(url[2]==='head'){
        this.head(url[3])
      }else if(url[2]==='kwords'){
        this.kwords(url[3])
      }else if(url[2]==='sex'){
        this.axios.get('/product/sex',{params:{
          sex:url[3]
        }}).then(res=>{
          this.productList=res.data;
        })
      }else if(url[2]==='size'){
        this.axios.get('/product/size',{params:{
          size:url[3]
        }}).then(res=>{
          this.productList=res.data;
        })
      }else if(url[2]==='brand'){
         this.axios.get('/product/brand',{params:{
          brand:url[3]
        }}).then(res=>{
          this.productList=res.data;
        })
      }else if(url[2]==='color'){
        this.axios.get('/product/color',{params:{
          color:url[3]
        }}).then(res=>{
          this.productList=res.data;
        })
      }else{
        switch(url[3]){
        case '默认排序':
          this.axios.get('/product/default').then(res=>{
            this.productList=res.data;
          })
          break;
        case '从低到高':
          this.axios.get('/product/asc').then(res=>{
            this.productList=res.data;
          })
          break;
        default:
          this.axios.get('/product/desc').then(res=>{
            this.productList=res.data;
          })
        }
      }
    }
  }
}

</script>

<style scoped>
.product{
  width: 1200px;
  margin: 0 auto 30px;
}
.product>.bread{
  margin: 10px 0;
  text-align: left;
}
.product>.container>.left{
  width: 300px;
}
.product>.container{
  display: flex;
}
.product>.container>.left>.price>h3{
  text-align: left;
  line-height: 40px;
}
.product>.container>.left>.price>.price-box{
  display: flex;
  justify-content: space-around;
  align-items: center;
  padding: 10px 0 20px;
  border-bottom: 1px solid #777;
}
.product>.container>.left>.price>div>input{
  width: 50px;
  text-align: center;
}
.product>.container>.left>.price>div>button{
  background-color: #fff;
  border: 1px solid #aaa;
  height: 24px;
  cursor: pointer;
  color: #777;
}
.product>.container>.left>.content>.top{
  display: flex;
  justify-content: space-between;
  line-height: 50px;
}
.product>.container>.left>.content>.top>span{
  font-size: 16px;
  cursor: pointer;
}
.product>.container>.left>.content>.list>.list-box{
  position: relative;
  height: 90px;
  overflow: hidden;
  overflow-y: auto;
}
/* 滚动条整体宽度 */
.product>.container>.left>.content>.list>.list-box::-webkit-scrollbar{
  width: 2px;
}
/* 轨道 */
.product>.container>.left>.content>.list>.list-box::-webkit-scrollbar-track{
  background-color: #888;
}
/* 滑块 */
.product>.container>.left>.content>.list>.list-box::-webkit-scrollbar-thumb{
  border-radius: 5px;
  background-color: #000;
}
/* .product>.container>.left>.content>.list>.list-box>.scorll-bar{
  position: absolute;
  width: 2px;
  height: 100%;
  background-color: #888;
  right: 0;
  top: 0;
} */
/* .product>.container>.left>.content>.list>.list-box>.scorll{
  position: absolute;
  width: 2px;
  height: 30px;
  background-color: #000;
  right: 0;
  top: 0;
} */
.product>.container>.left>.content>.list>.list-box>div{
  margin-left: 30px;
  text-align: left;
}

.product>.container>.left>div:last-child{
  height: 200px;
  position: relative;
  z-index: 1;
  background: #fff;
}
.product>.container>.left>.content>.list>.list-box>div>a>.look{
  display: inline-block;
  width: 8px;
  height: 8px;
  margin-right: 10px; 
  border: 1px solid #000;
  border-radius: 3px;
  transition: .5s linear;
}
.product>.container>.left>.content>.list>.list-box>div>a{
  padding: 6px 0;
}
.product>.container>.left>.content>.list>.list-box>div>.router-link-active{
  border-bottom: 1px solid #000;
}
.product>.container>.left>.content>.list{
  height: 0px;
  border-bottom: 1px solid #777;
  overflow: hidden;
  opacity: .3;
  transition: .5s linear;
}
.product>.container>.left>.content>.active{
  height: 100px;
  border-bottom: 1px solid #777;
  background-color: #fff;
  opacity: 1;
}
.product>.container>.left>.content>.list>.list-box>div>a>.title{
  line-height: 30px;
  color: #777;
  transition: .5s linear;
  cursor: pointer;
}
.product>.container>.left>.content>.list>.list-box>div:hover .look{
  background-color: #000; 
}
.product>.container>.right>.right-top{
  display: flex;
  flex-wrap: wrap;
  width: 900px;
  margin-bottom: 20px;
}
.product>.container>.right>.right-top>.list{
  position: relative;
  width: 300px;
  height: 300px;
  margin-bottom: 10px;
}
.product>.container>.right>.right-top>.list>a>img{
  width: 200px;
  height: 200px;
  margin-top: 10px;
}
.product>.container>.right>.right-top>.list>p{
  line-height: 20px;
}
.product>.container>.right>.right-top>.list>hr{
  width: 50px;
}
.product>.container>.right>.right-top>.list>.spec{
  position: absolute;
  top: 0;
  right: -80px;
  width: 0;
  height: 60%;
  background-color: #fff;
  opacity: .7;
  transition: 1s;
  overflow: hidden;
}
.product>.container>.right>.right-top>.list>.spec>p{
  width: 80px;
}
.product>.container>.right>.right-top>.list:hover .spec{
  position: absolute;
  top: 0;
  right: 10px;
  width: 80px;
}
.product>.container>.right>h1{
  line-height: 200px;
  opacity: .7;
}
</style>