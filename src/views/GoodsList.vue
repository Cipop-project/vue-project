<template>
  <div>
    <nav-header></nav-header>
    <nav-bread>
      <!--插槽使用-->
      <span>Goods</span>
    </nav-bread>
    <div class="accessory-result-page accessory-page">
      <div class="container">
        <div class="filter-nav">
          <span class="sortby">Sort by:</span>
          <a href="javascript:void(0)" class="default cur">Default</a>
          <a @click="sortGoods" href="javascript:void(0)" class="price">
            Price
            <svg class="icon icon-arrow-short" v-bind:class="{'sort-up':!sortFlag}">
              <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#icon-arrow-short"></use>
            </svg>
          </a>
          <a href="javascript:void(0)" class="filterby stopPop" @click="showFilterPop">Filter by</a>
        </div>
        <div class="accessory-result">
          <!-- filter -->
          <div class="filter stopPop" id="filter" v-bind:class="{'filterby-show':filterBy}">
            <dl class="filter-price">
              <!--特殊存在未绑定-->
              <dt>Price:</dt>
              <dd><a href="javascript:void(0)"  v-bind:class="{'cur':priceChecked=='all'}" @click="setPriceFilter('all')">All</a>
              </dd>
              <!--v-for 绑定格式相同数据-->
              <dd v-for="(prices,index) in priceFilter" :key='"priceFilter"+index'>
                <a href="javascript:void(0)" @click="setPriceFilter(index)" v-bind:class="{'cur':priceChecked==index}">{{prices.startPrice}}
                  - {{prices.endPrice}}</a>
              </dd>
            </dl>
          </div>

          <!-- search result accessories list -->
          <div class="accessory-list-wrap">
            <div class="accessory-list col-4">
                            <ul>
                              <li v-for="(item,index) in goodsList" :key='"goodsList"+index'>
                                <div class="pic">
                                  <!--图片需要动态绑定，不能直接写src=""，这样会因页面渲染太快而导致图片未加载-->
                                 <a href="#"><img v-lazy=" 'static/' + item.productImage" :key='"img"+item.productImage' ></a>
                                </div>
                                <div class="main">
                                  <div class="name">{{item.productName}}</div>
                                  <div class="price">{{item.salePrice}}</div>
                                  <div class="btn-area">
                                    <a href="javascript:;" class="btn btn--m" @click="addCart(item.productId)">加入购物车</a>
                                  </div>
                                </div>
                              </li>
                            </ul>
                <!-- 滚动条 -->
                <div class="load-more" v-infinite-scroll="loadMore" infinite-scroll-disabled="busy" infinite-scroll-distance="10">
                    <img v-show="isLoading" src="static/loading-svg/loading-balls.svg"/>
                </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!--价格选择遮罩层-->
    <div class="md-overlay" v-show="overLayFlag" @click="closePop"></div>
    <!-- 未登录，禁止加入购物车提示 -->
    <model v-bind:mdShow="mdShow" v-on:close="closeModel()">
      <p slot="message">
        请先登录，否则无法加入购物车！
      </p>
      <div slot="btnGroup">
        <a class="btn btn--m" href="javascript:;" @click="mdShow=false">
          关闭
        </a>
      </div>
    </model>
    <!-- 加入购物车成功提示 -->
    <model v-bind:mdShow="mdShowCart" v-on:close="closeModel()">
      <p slot="message">
          <svg class="icon-status-ok">
            <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#icon-status-ok"></use>
          </svg>
          <span>加入购物车成功！</span>
      </p>
      <div slot="btnGroup">
        <a class="btn btn--m" href="javascript:;" @click="mdShowCart=false">
          继续购物
        </a>
        <router-link class="btn btn--m" href="javascript:;" to="/cart">进入购物车查看</router-link>
      </div>
    </model>
    <nav-footer></nav-footer>
  </div>
</template>

<script>
//引用css静态资源
import "./../assets/css/base.css";
import "./../assets/css/product.css";
import NavHeader from "@/components/NavHeader.vue";
import NavFooter from "@/components/NavFooter.vue";
import NavBread from "@/components/NavBread.vue";
import Model from "@/components/Model";
import axios from "axios";

export default {
  data() {
    return {
      //商品数据数组
      goodsList: [],
      //价格条件
      priceFilter: [
        {
          startPrice: "0.00",
          endPrice: "100.00"
        },
        {
          startPrice: "100.00",
          endPrice: "500.00"
        },
        {
          startPrice: "500.00",
          endPrice: "1000.00"
        },
        {
          startPrice: "1000.00",
          endPrice: "5000.00"
        }
      ],
      //价格条件被选中
      priceChecked: "all",
      //价格选择是否隐藏
      filterBy: false,
      //遮罩层
      overLayFlag: false,
      //排序
      sortFlag: true,
      page: 1,
      pageSize: 8,
      //滚动条
      busy: true,
      //正在加载
      isLoading: false,
      //未登录，禁止加入购物车Model框
      mdShow: false,
      //加入购物车成功
      mdShowCart:false
    };
  },
  components: {
    NavHeader,
    NavFooter,
    NavBread,
    Model
  },
  mounted: function() {
    //获取商品数据
    this.getGoodsList();
  },
  methods: {
    //获取商品数据
    getGoodsList(addFlag) {
      var param = {
        page: this.page,
        pageSize: this.pageSize,
        sort: this.sortFlag ? 1 : -1,
        priceLevel: this.priceChecked
      };
      this.isLoading = true;
      axios
        .get("/goods", {
          params: param
        })
        .then(res => {
          this.isLoading = false;
          //console.log(res);
          var data = res.data;
          if (addFlag) {
            //滚动时商品需要累加
            this.goodsList = this.goodsList.concat(data.result.list);
            //无返回商品则不滚动
            if (data.result.count < this.pageSize) {
              this.busy = true;
            } else {
              this.busy = false;
            }
          } else {
            this.goodsList = data.result.list;
            this.busy = false;
          }
        });
    },
    //商品排序
    sortGoods() {
      this.sortFlag = !this.sortFlag;
      this.page = 1;
      this.getGoodsList();
    },
    //响应式布局：点击显示价格
    showFilterPop() {
      this.filterBy = true;
      this.overLayFlag = true;
    },
    //价格选择：菜单样式改变；响应式时：关闭价格选择，关闭遮罩层
    setPriceFilter(index) {
      this.priceChecked = index;
      this.closePop();
      this.page = 1;
      this.getGoodsList();
    },
    //滚动条
    loadMore: function() {
      this.busy = true;
      setTimeout(() => {
        this.page++;
        this.getGoodsList(true);
      }, 500);
    },
    //响应式布局：点击关闭价格选择和遮罩层
    closePop() {
      this.filterBy = false;
      this.overLayFlag = false;
    },
    //加入购物车
    addCart(productId) {
      axios
        .post("/goods/addCart", {
          productId: productId
        })
        .then(res => {
          if (res.data.status == 0) {
            this.mdShowCart = true
          } else {
            this.mdShow = true;
          }
        });
    },
    //关闭模态框
    closeModel() {
      this.mdShow = false;
      this.mdShowCart = false;
    }
  }
};
</script>

<style scoped>
.load-more {
  height: 100px;
  line-height: 100px;
  text-align: center;
}

.btn:hover {
  background-color:rgb(218, 139, 139);
  transition: all 0.3s ease-out;
}

.sort-up {
  transform: rotate(180deg);
  transition: all 0.3s ease-out;
}

.icon-arrow-short {
  transition: all 0.3s ease-out;
}
</style>
