<template>
    <div class="address">
        <Header title="选择收货地址" :isLeft="true"/>
        <div class="city_search">
            <div class="search">
                <span class="city" @click="$router.push('city')">
                    {{city}}
                    <i class="fa fa-angle-down"></i>
                </span>
                <i class="fa fa-search"></i>
                <input type="text" v-model="search_val" placeholder="小区/写字楼/学校等">
            </div>
            <location @click="selectAddress" :address="address"/>
        </div>
        <div class="area">
            <ul class="area_list" v-for="(item,index) of area_list" :key="index">
                <li @click="selectAddress(item)">
                    <h4>{{item.name}}</h4>
                    <p>{{item.district}}{{item.address}}</p>
                </li>
            </ul>
        </div>
    </div>
</template>

<script>
    import Header from '../components/Header'
    import Location from '../components/Location'

    export default {
        name: "Address",
        components: {
            Header,
            Location
        },
        computed: {
            address() {
                return this.$store.getters.location.formattedAddress
            }
        },
        watch: {
            search_val() {
                this.searchPlace();
            }
        },
        data() {
            return {
                city: "未获取城市", //当前的城市
                search_val: "",//搜索框中的值
                area_list: []//从高德的组件中获取的搜索建议列表
            }
        },
        beforeRouteEnter(to, from, next) { //这里接收上级路由（主页）和下级路由（城市选择）传来的city参数
            next(vm => {
                vm.city = to.params.city;
            });
        },
        methods: {
            searchPlace() {
                const self = this;
                // console.log(this.search_val)
                var map = new AMap.Map('container', {
                    resizeEnable: true
                });
                map.plugin('AMap.Autocomplete', function () {
                    // 实例化Autocomplete
                    var autoOptions = {
                        //city 限定城市，默认全国
                        city: self.city
                    }
                    var autoComplete = new AMap.Autocomplete(autoOptions);
                    autoComplete.search(self.search_val, function (status, result) {
                        // 搜索成功时，result即是对应的匹配数据
                        self.area_list = result.tips;
                    });
                })
            },
            selectAddress(item) {
                if (item) {
                    //设置地址
                    this.$store.dispatch('setAddress', item.district + item.address + item.name);
                }else{ //这里触发条件是用户在选择收货地址时点击了原来的定位，所以不用传新参
                    this.$store.dispatch('setAddress',this.address)
                }
                // 选择完城市立马跳转回home
                this.$router.push('/home');
            }
        }
    }
</script>

<style scoped>
    .address {
        width: 100%;
        height: 100%;
        overflow: auto;
        box-sizing: border-box;
        padding-top: 45px;
    }

    .city_search {
        background-color: #fff;
        padding: 10px 20px;
        color: #333;
    }

    .search {
        background-color: #eee;
        height: 40px;
        border-radius: 10px;
        box-sizing: border-box;
        line-height: 40px;
    }

    .search .city {
        padding: 0 10px;
    }

    .city i {
        margin-right: 10px;
    }

    .search input {
        margin-left: 5px;
        background-color: #eee;
        outline: none;
        border: none;
    }

    .area {
        margin-top: 16px;
        background: #fff;
    }

    .area li {
        border-bottom: 1px solid #eee;
        padding: 8px 16px;
        color: #aaa;
    }

    .area li h4 {
        font-weight: bold;
        color: #333;
        margin-bottom: 5px;
    }
</style>
