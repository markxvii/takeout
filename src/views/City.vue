<template>
    <div class="city">
        <div class="search_wrap">
            <div class="search">
                <i class="fa fa-search"></i>
                <input v-model="city_val" type="text" placeholder="输入城市名">
            </div>
            <button @click="$router.push({name:'address',params:{city:city}})">取消</button>
        </div>
        <div style="height: 100%" v-if="searchList.length===0">
            <div class="location">
                <!--                这里为啥要传name:city，因为接收的函数中的city包含name属性，这里是纯字符，所以构造一个假对象-->
                <Location @click="selectCity({name:city})" :address="city"/>
            </div>
            <Alphabet @selectCity="selectCity" ref="allCity" :cityInfo="cityInfo" :keys="keys"/>
        </div>
        <div class="search_list" v-else>
            <ul>
                <li v-for="(item,index) of searchList" :key="index" @click="selectCity(item)">
                    {{item.name}}
                </li>
            </ul>
        </div>
    </div>
</template>

<script>
    import Location from '../components/Location'
    import Alphabet from '../components/Alphabet'

    export default {
        name: "City",
        data() {
            return {
                city_val: "", //input双向绑定
                cityInfo: null, //总数组
                keys: [],  //字母表
                allCities: [], //所有城市名
                searchList: [] //搜索时建议列表
            }
        },
        components: {
            Location,
            Alphabet
        },
        watch: {
            //监听input输入框
            city_val(val) {
                this.searchCity();
            }
        },
        computed: {
            city() {
                return this.$store.getters.location.addressComponent.city || this.$store.getters.location.addressComponent.province
            }
        },
        created() {
            this.getCityInfo();
        },
        methods: {
            searchCity() {
                if (!this.city_val) {
                    this.searchList = [];
                } else {
                    this.searchList = this.allCities.filter(item => {
                        return item.name.indexOf(this.city_val) !== -1;
                    })
                }
            },
            getCityInfo() {
                this.$axios('/api/posts/cities').then(res => {
                    // console.log(res.data);
                    this.cityInfo = res.data;
                    //处理key
                    this.keys = Object.keys(res.data);
                    //移除掉接口返回的最后一个不需要的key
                    this.keys.pop();
                    //keys排序
                    this.keys.sort();
                    //等待数据读取完毕立即更新子组件
                    this.$nextTick(() => {
                        this.$refs.allCity.initScroll();
                    });
                    //获取所有城市名列表，用于搜索
                    this.keys.forEach(key => {  //遍历字母表
                        this.cityInfo[key].forEach(city => {  //遍历所有数据表
                            this.allCities.push(city);
                        })
                    })
                }).catch(err => {
                    console.log(err)
                });
            },
            selectCity(city) {
                this.$router.push({name: "address", params: {city: city.name}})
            }
        }
    }
</script>

<style scoped>
    .city {
        width: 100%;
        height: 100%;
        overflow: auto;
        box-sizing: border-box;
        padding-top: 45px;
    }

    .search_wrap {
        position: fixed;
        top: 0;
        height: 45px;
        width: 100%;
        background: #fff;
        box-sizing: border-box;
        padding: 3px 16px;
        display: flex;
        justify-content: space-between;
    }

    .search {
        background-color: #eee;
        border-radius: 10px;
        line-height: 40px;
        width: 85%;
        box-sizing: border-box;
        padding: 0 16px;
    }

    .search input {
        background: #eee;
        outline: none;
        border: none;
        margin-left: 5px;
    }

    .search_wrap button {
        outline: none;
        border: none;
        color: #009eef;
        background: transparent;
        font-weight: bold;
    }

    .location {
        background: #fff;
        padding: 8px 16px;
        height: 65px;
        box-sizing: border-box;
    }

    .search_list {
        background: #fff;
        padding: 5px 16px;
    }

    .search_list li {
        padding: 10px;
        border-bottom: 1px solid #eee;
    }
</style>
