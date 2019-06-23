<template>
    <div :class="{'open':isSort || isScreen}" @click.self="hideView">
        <!--        导航-->
        <div v-if="filterData" class="filter_wrap">
            <aside class="filter">
                <div class="filter-nav" v-for="(item,index) of filterData.navTab" :key="index"
                     :class="{'filter-bold':currentFilter===index}"
                     @click="filterSort(index)"
                >
                    <span>{{item.name}}</span>
                    <i v-if="item.icon" :class="'fa fa-'+item.icon"></i>
                </div>
            </aside>
        </div>
        <!--        排序-->
        <section class="filter-extend" v-if="isSort">
            <ul>
                <li v-for="(item,index) of filterData.sortBy" :key="index" @click="selectSort(item,index)">
                    <span :class="{'selectName':currentSort===index}">{{item.name}}</span>
                    <i class="fa fa-check" v-show="currentSort===index"></i>
                </li>
            </ul>
        </section>
        <!--        筛选-->
        <section class="filter-extend" v-if="isScreen">
            <div class="filter-sort">
                <div v-for="(screen,index) of filterData.screenBy" :key="index" class="morefilter">
                    <p class="title">{{screen.title}}</p>
                    <ul>
                        <li v-for="(item,i) of screen.data" :key="i" :class="{'selected':item.select}"
                            @click="selectScreen(item,screen)">
                            <img v-if="item.icon" :src="item.icon" alt="">
                            <span>{{item.name}}</span>
                        </li>
                    </ul>
                </div>
            </div>
            <div class="morefilter-btn">
                <span @click="clearFilter" class="modrefilter-clear" :class="{'edit':edit}">清空</span>
                <span class="morefilter-ok" @click="filterOK">确定</span>
            </div>
        </section>
    </div>
</template>

<script>
    export default {
        name: "FilterView",
        data() {
            return {
                currentFilter: 0, //当前选择的4种排序方式的下标
                isSort: false, //是否是综合排序
                currentSort: 0, //综合排序中当前选择的
                isScreen: false,//是否是筛选
            };
        },
        computed: {
            edit() {
                let edit = false;
                this.filterData.screenBy.forEach(screen => {
                    screen.data.forEach(item => {
                        if (item.select) {
                            edit = true;
                        }
                    })
                });
                return edit;
            }
        },
        props: {
            filterData: Object
        },
        methods: {
            filterSort(index) {
                this.currentFilter = index;
                switch (index) {
                    case 0: //综合排序
                        this.isSort = true;
                        this.$emit('searchFixed', true);
                        break;
                    case 1: //距离最近
                        this.$emit('update', {condition: this.filterData.navTab[1].condition}); //相当于综合排序里面的code值
                        this.hideView();
                        break;
                    case 2: //品质联萌
                        this.$emit('update', {condition: this.filterData.navTab[2].condition}); //相当于综合排序里面的code值
                        this.hideView();
                        break;
                    case 3: //筛选
                        this.isScreen = true;
                        this.isSort = false;
                        this.$emit('searchFixed', true);
                        break;
                    default:
                        this.hideView();
                        break;
                }
            },
            hideView() {
                //在点击遮罩处调用
                this.isSort = false;
                this.isScreen = false;
                this.$emit('searchFixed', false);
            },
            //点击综合排序中的项目更新currentSort
            selectSort(item, index) {
                this.currentSort = index;
                this.filterData.navTab[0].name = this.filterData.sortBy[index].name;//修改筛选栏第一处的名字为你点击的名字
                this.hideView();

                //更新数据
                this.$emit('update', {condition: item.code})//每个遍历出来的综合排序下的item拥有一个独立的code用于区分
            },
            selectScreen(item, screen) {
                if (screen.id !== "MPI") {
                    //MPI代表商家服务，下面的按钮即为单选
                    screen.data.forEach(ele => {
                        ele.select = false; //因为是单选所以其他每个元素都要变成false，如果是商家服务则跳过这一步，即为多选
                    })
                }
                item.select = !item.select;//点击的这一项取反
            },
            clearFilter() {
                this.filterData.screenBy.forEach(screen => {
                    screen.data.forEach(item => {
                        item.select = false
                    })
                })
            },
            filterOK() {
                let screenData = {
                    MPI: '',
                    offer: '',
                    per: ''
                }
                let mpiStr = "";
                this.filterData.screenBy.forEach(screen => {
                    screen.data.forEach((item, index) => {
                        if (item.select) {
                            //多选或单选
                            if (screen.id !== "MPI") {
                                // 单选
                                //todo:接口没有
                            } else {
                                //多选
                                mpiStr += item.code + ",";
                                screenData[screen.id] = mpiStr; //screen.id就是MPI/OFFER/PER
                            }
                        }
                    });
                });
                // console.log(mpiStr);
                this.$emit('update', {condition: screenData});
                this.hideView();
            }
        }
    }
</script>

<style scoped>
    .filter_wrap {
        background: #fff;
        position: sticky;
        top: 54px;
        z-index: 10;
    }

    .filter {
        position: relative;
        border-bottom: 1px solid #ddd;
        line-height: 10.4vw;
        z-index: 101;
        height: 10.666667vw;
        display: flex;
        justify-content: space-around;
    }

    .filter-nav {
        flex: 1;
        text-align: center;
        color: #666;
        font-size: 0.8333rem;
    }

    .filter-nav i {
        width: 1.6vw;
        height: 0.8vw;
        margin-left: 1.333333vw;
        margin-bottom: 0.533333vw;
        fill: #333;
        will-change: transform;
    }

    .filter-bold {
        font-weight: 600;
        color: #333;
    }

    .open {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.5);
        transition: all 0.3s ease-in-out;
        z-index: 3;
    }

    .filter-extend {
        background-color: #fff;
        color: #333;
        padding-top: 2.133333vw;
        position: absolute;
        width: 100%;
        z-index: 4;
        left: 0;
        top: 24.533333vw;
    }

    .filter-extend li {
        position: relative;
        padding-left: 5.333333vw;
        line-height: 10.666667vw;
        overflow: hidden;
    }

    .fa-check {
        float: right;
        color: #009eef;
        margin-right: 3.733333vw;
        line-height: 10.666667vw;
    }

    .selectName {
        color: #009eef;
    }

    /* 筛选 */
    .filter-sort {
        background: #fff;
        padding: 0 2.666667vw;
        line-height: normal;
    }

    .morefilter {
        margin: 2.666667vw 0;
        overflow: hidden;
    }

    .morefilter .title {
        margin-bottom: 2vw;
        color: #666;
        font-size: 0.5rem;
    }

    .morefilter ul {
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
        font-size: 0.8rem;
    }

    .morefilter li {
        box-sizing: border-box;
        width: 30%;
        height: 9.333333vw;
        line-height: 9.333333vw;
        margin: 0.8vw 1%;
        background: #fafafa;
    }

    .morefilter li img {
        width: 3.466667vw;
        height: 3.466667vw;
        vertical-align: middle;
        margin-right: 0.8vw;
    }

    .morefilter li span {
        margin-left: 2%;
        vertical-align: middle;
    }

    .morefilter-btn {
        display: flex;
        justify-content: space-around;
        align-items: center;
        background-color: #fafafa;
        box-shadow: 0 -0.266667vw 0.533333vw 0 #ededed;
        line-height: 11.466667vw;
        box-sizing: border-box;
    }

    .morefilter-btn span {
        font-size: 0.826667rem;
        text-align: center;
        text-decoration: none;
        flex: 1;
    }

    .morefilter-clear {
        color: #ddd;
        background: #fff;
    }

    .morefilter-ok {
        color: #fff;
        background: #00d762;
        border: 0.133333vw solid #00d762;
    }

    .selected {
        color: #3190e8 !important;
        background-color: #edf5ff !important;
    }

    .edit {
        color: #333 !important;
    }
</style>
