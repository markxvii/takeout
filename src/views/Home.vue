<template>
    <div class="home">
        <div class="header">
            <div class="address_map" @click="$router.push({name: 'address',params: {city: city}})">
                <i class="fa fa-map-marker"></i>
                <span>{{address}}</span>
                <i class="fa fa-sort-desc"></i>
            </div>
        </div>
        <div class="search_wrap" :class="{'fixedview':showFilter}" @click="$router.push('/search')">
            <div class="shop_search">
                <i class="fa fa-search"></i>
                搜索商家 商家名称
            </div>
        </div>
        <div id="container">
            <!--            轮播图-->
            <mt-swipe :auto="4000" class="swiper">
                <mt-swipe-item v-for="(img,index) of swipeImgs" :key="index">
                    <img :src="img" alt="">
                </mt-swipe-item>
            </mt-swipe>
            <!--            商品分类-->
            <mt-swipe :auto="0" class="entries">
                <mt-swipe-item v-for="(entry,i) of entries" :key="i" class="entry_wrap">
                    <div v-for="(item,index) of entry" :key="index" class="foodentry">
                        <div class="img_wrap">
                            <img :src="item.image" alt="">
                        </div>
                        <span>{{item.name}}</span>
                    </div>
                </mt-swipe-item>
            </mt-swipe>
        </div>
        <div class="shoplist-title">推荐商家</div>

        <!--        筛选导航-->
        <FilterView :filterData="filterData" @searchFixed="showFilterView" @update="update"/>

        <!--        商家信息-->
        <mt-loadmore :auto-fill="false" :bottom-pull-text="bottomPullText" :top-method="loadData"
                     :bottom-method="loadMore" :bottom-all-loaded="allLoaded" ref="loadmore">
            <div class="shoplist">
                <IndexShop v-for="(item,index) of restaurants" :key="index" :restaurant="item.restaurant"/>
            </div>
        </mt-loadmore>

    </div>
</template>

<script>
    import {Swipe, SwipeItem, Loadmore} from 'mint-ui';
    import FilterView from '../components/FilterView'
    import IndexShop from '../components/IndexShop'

    export default {
        name: "Home",
        data() {
            return {
                swipeImgs: [],
                entries: [],
                filterData: null,
                showFilter: false,
                page: 1,
                size: 5,
                restaurants: [], //所有商家
                allLoaded: false,//所有数据是否已经加载完，和页码有关，并不是数据的加载状态
                bottomPullText: '上拉加载更多',
                data:null
            };
        },
        components: {
            Swipe,
            SwipeItem,
            FilterView,
            IndexShop
        },
        computed: {
            address() {
                return this.$store.getters.address;
            },
            city() {
                return this.$store.getters.location.addressComponent.city || this.$store.getters.location.addressComponent.province
            }
        },
        created() {
            this.getData();
        },
        methods: {
            //mintui下拉刷新
            loadData() {
                //重置页码数据
                this.page = 1;
                this.allLoaded = false;
                this.bottomPullText = '上拉加载更多'
                //获取商家信息
                this.$axios.post(`api/profile/restaurants/${this.page}/${this.size}`,this.data)
                    .then(res => {
                        // console.log(res.data);
                        this.$refs.loadmore.onTopLoaded();//下拉刷新完毕后执行，列表恢复正常
                        this.restaurants = res.data;
                    });
            },
            //mintui上拉加载
            loadMore() {
                if (!this.allLoaded) { //如果还有数据没加载完
                    this.page++;
                    //获取商家信息
                    this.$axios.post(`api/profile/restaurants/${this.page}/${this.size}`)
                        .then(res => {
                            //加载完之后重新渲染
                            this.$refs.loadmore.onBottomLoaded();
                            if (res.data.length > 0) { //如果接口还有数据提供
                                res.data.forEach(item => {
                                    this.restaurants.push(item);
                                });
                                if (res.data.length < this.size) { //但如果接口返回数据小于5条
                                    this.allLoaded=true;
                                    this.bottomPullText='没有更多了哦'
                                }
                            }else{
                                //数据为空
                                this.allLoaded = true;
                                this.bottomPullText="没有更多了哦"
                            }
                        });
                }
            },
            getData() {
                //请求首页图片、美食分类
                this.$axios('/api/profile/shopping')
                    .then(res => {
                        // console.log(res);
                        this.swipeImgs = res.data.swipeImgs; //首页图片
                        this.entries = res.data.entries;//美食分类
                    });

                //获取筛选列表
                this.$axios('/api/profile/filter')
                    .then(res => {
                        this.filterData = res.data;
                    });

                this.loadData();
            },
            showFilterView(isShow) {
                this.showFilter = isShow;
            },
            //这里从筛选组件中接收综合排序中的具体排序方式
            update(condition) {
                this.data = condition;
                this.loadData();
            }
        }
    }
</script>

<style scoped>
    .home {
        width: 100%;
        height: 100%;
        overflow: auto;
        box-sizing: border-box;
    }

    .header, .search_wrap {
        background-color: #009eef;
        padding: 10px 16px;
    }

    .header .address_map {
        color: #fff;
        font-weight: bold;
    }

    .address_map i {
        margin: 0 3px;
        font-size: 18px;
    }

    .address_map span {
        display: inline-block;
        width: 80%;
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
    }

    .search_wrap .shop_search {
        background-color: #fff;
        padding: 10px 0;
        border-radius: 4px;
        text-align: center;
        color: #aaa;
    }

    .search_wrap {
        position: sticky;
        top: 0px;
        z-index: 999;
        box-sizing: border-box;

    }

    .swiper {
        height: 100px;
    }

    .swiper img {
        width: 100%;
        height: 100px;
    }

    .entries {
        background: #fff;
        height: 47.2vw;
        text-align: center;
        overflow: hidden;
    }

    .foodentry {
        width: 20%;
        float: left;
        position: relative;
        margin-top: 2.933333vw;
    }

    .foodentry .img_wrap {
        position: relative;
        display: inline-block;
        width: 12vw;
        height: 12vw;
    }

    .img_wrap img {
        width: 100%;
        height: 100%;
    }

    .foodentry span {
        display: block;
        color: #666;
        font-size: 0.32rem;
    }

    /* 推荐商家 */
    .shoplist-title {
        display: flex;
        align-items: flex;
        justify-content: center;
        height: 9.6vw;
        line-height: 9.6vw;
        font-size: 16px;
        color: #333;
        background: #fff;
    }

    .shoplist-title:after,
    .shoplist-title:before {
        display: block;
        content: "一";
        width: 5.333333vw;
        height: 0.266667vw;
        color: #999;
    }

    .shoplist-title:before {
        margin-right: 3.466667vw;
    }

    .shoplist-title:after {
        margin-left: 3.466667vw;
    }

    .fixedview {
        width: 100%;
        position: fixed;
        top: 0;
        z-index: 999;
    }

    .mint-loadmore {
        height: calc(100% - 95px);
        overflow: auto;
    }
</style>
