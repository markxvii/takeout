<template>
    <div class="addAddress">
        <Header :isLeft="true" :title="title"/>
        <div class="viewbody">
            <div class="content">
                <FormBlock label="联系人" placeholder="姓名" :tags="sex" @checkSex="checkSex" :sex="addressInfo.sex" v-model="addressInfo.name"/>
                <FormBlock label="电话" placeholder="手机号码" v-model="addressInfo.phone"/>
                <FormBlock label="地址" placeholder="小区/写字楼/学校等" icon="angle-right" @click="showSearch=true" v-model="addressInfo.address"/>
                <FormBlock label="门牌号" placeholder="10号楼" icon="edit" textarea="textarea" v-model="addressInfo.bottom"/>
                <div class="formblock">
                    <div class="label-wrap">标签</div>
                    <TabTag :tags="tags" :selectedTag="addressInfo.tag" @checkTag="checkTag"/>
                </div>
            </div>
            <div class="form-button-wrap">
                <button class="form-button" @click="handleSave">确定</button>
            </div>
        </div>
        <AddressSearch :addressInfo="addressInfo" :showSearch="showSearch" @close="showSearch=false"/>
    </div>
</template>

<script>
    import Header from '../../components/Header'
    import FormBlock from '../../components/Orders/FormBlock'
    import TabTag from '../../components/Orders/TabTag'
    import AddressSearch from '../../components/Orders/AddressSearch'
    import {Toast} from 'mint-ui';

    export default {
        name: "AddAddress",
        data() {
            return {
                title: '',
                tags: ['家', '学校', '公司'],
                sex: ["先生", '女士'],
                addressInfo:{}, //表单填写的所有项目,
                showSearch:false
            }
        },
        components: {
            Header,
            FormBlock,
            TabTag,
            AddressSearch
        },
        //接收初始化好的数据
        beforeRouteEnter(to, from, next) {
            next(vm=>{
                vm.addressInfo = to.params.addressInfo;
                vm.title=to.params.title;
            })
        },
        methods:{
            checkTag(i) {
                this.addressInfo.tag = i;
            },
            checkSex(i) {
                this.addressInfo.sex = i;
            },
            handleSave() {
                if (!this.addressInfo.name) {
                    this.showMsg('请输入联系人');
                    return;
                }
                if (!this.addressInfo.phone) {
                    this.showMsg('请输入手机');
                    return;
                }
                if (!this.addressInfo.address) {
                    this.showMsg('请输入收货地址');
                    return;
                }
                if (this.title==='添加地址'){
                    this.addAddress();
                }else{
                    this.editAddress();
                }
            },
            addAddress() {
                this.$axios.post(`/api/user/add_address/${localStorage.ele_login}`,this.addressInfo)
                    .then(res=>{
                        if (!this.$store.getters.userInfo) {
                            this.$store.dispatch('setUserInfo', this.addressInfo);
                        }
                        this.$router.push('myAddress');
                    })
                    .catch(err=>{console.log(err)})
            },
            editAddress() {
                this.$axios.post(`/api/user/edit_address/${localStorage.ele_login}/${this.addressInfo._id}`,this.addressInfo)
                    .then(res=>{
                        this.$router.push("/myAddress");
                    })
            },
            showMsg(msg) {
                Toast({
                    message:msg,
                    postion:'bottom',
                    duration:2000
                })
            }
        }
    }
</script>

<style scoped>
    .addAddress {
        width: 100%;
        height: 100%;
        box-sizing: border-box;
        padding-top: 45px;
    }

    .viewbody {
        width: 100%;
        height: 100%;
        overflow: auto;
        box-sizing: border-box;
        background-color: #f5f5f5;
    }

    .content {
        padding-left: 4vw;
        background: #fff;
        font-size: 0.94rem;
    }

    .formblock {
        background-color: #fff;
        border-bottom: 1px solid #eee;
        display: flex;
    }

    .formblock .label-wrap {
        flex-basis: 17.333333vw;
        padding: 3.733333vw 0;
        line-height: 4.8vw;
        color: #333;
        font-weight: 700;
    }

    /* 确定按钮 */
    .form-button-wrap {
        padding: 5.333333vw 4vw;
        display: flex;
    }

    .form-button-wrap .form-button {
        background: #00d762;
        text-align: center;
        border-radius: 0.533333vw;
        flex: 1;
        font-size: 1.1rem;
        line-height: 5.066667vw;
        color: #fff;
        padding: 3.333333vw 0;
        border: none;
        font-weight: 500;
    }
</style>
