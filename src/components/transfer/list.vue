<template>
    <div :class="classes" :style="listStyleFilter">
        <div :class="prefixCls + '-header'">
            <Checkbox :value="checkedAll" :disabled="checkedAllDisabled" @on-change="toggleSelectAll"></Checkbox>
            <span :class="prefixCls + '-header-title'" @click="toggleSelectAll(!checkedAll)">{{ title }}</span>
            <span :class="prefixCls + '-header-count'">{{ count }}</span>
        </div>
        <div :class="bodyClasses">
            <div :class="prefixCls + '-body-search-wrapper'" v-if="filterable">
                <Search
                    :prefix-cls="prefixCls + '-search'"
                    :query="query"
                    @on-query-clear="handleQueryClear"
                    @on-query-change="handleQueryChange"
                    :placeholder="filterPlaceholder"></Search>
            </div>
            <ul :class="prefixCls + '-content'" style="position: relative">
                <li
                    v-for="item in filterData"
                    :class="itemClasses(item)"
                    @click.prevent="select(item)">
                    <Checkbox :value="isCheck(item)" :disabled="item.disabled"></Checkbox>
                    <span v-html="showLabel(item)"></span>
                </li>
                <li :class="prefixCls + '-content-not-found'">{{ notFoundText }}</li>
                <li class="ivu-transfer-split-page" v-if="this.splitPage">
                    <div><span class="right" @click="lastPage(0)">首页</span><span @click="lastPage(1)">上一页</span></div>
                    <div><input v-model="page"/></div>
                    <div><span @click="nextPage(1)" class="right">下一页</span><span @click="nextPage(0)">尾页</span></div>
                </li>
            </ul>
        </div>
        <div :class="prefixCls + '-footer'" v-if="showFooter"><slot></slot></div>
    </div>
</template>
<script>
    import Search from './search.vue';
    import Checkbox from '../checkbox/checkbox.vue';

    export default {
        name: 'TransferList',
        components: { Search, Checkbox },
        props: {
            //分页数量
            splitPage: {
                type: Number,
                default: 0
            },
            //总页数
            page:{
                type:Number,
                default: 1
            },
            prefixCls: String,
            data: Array,
            renderFormat: Function,
            checkedKeys: Array,
            listStyle: Object,
            title: [String, Number],
            filterable: Boolean,
            filterPlaceholder: String,
            filterMethod: Function,
            notFoundText: String,
            validKeysCount: Number
        },
        data () {
            return {
                showItems: [],
                query: '',
                showFooter: true
            };
        },
        watch: {
            data () {
                this.updateFilteredData();
            }
        },
        computed: {
            // searchPage:{
            //     get(){
            //         return this.page;
            //     },
            //     set(val){
            //         const page=val;
            //         const reg=/^\+?[1-9][0-9]*$/;//正整数
            //         const flag=reg.test(page);
            //         if (flag||page==0) {
            //             this.page=page;
            //         } else {
            //             this.page=1;
            //         }
            //         this.$forceUpdate();
            //     }
            // },
            listStyleFilter(){
                if (this.splitPage) {
                    let height;
                    if (this.filterable) {
                        height=105+(this.splitPage+1)*32.4;
                    } else {
                        height=75+(this.splitPage+1)*32.4;
                    }
                    let obj={height:`${height}px`,width:`${this.listStyle.width}px`};
                    return obj;
                } else {
                    return this.listStyle;
                }
            },
            classes () {
                return [
                    `${this.prefixCls}`,
                    {
                        [`${this.prefixCls}-with-footer`]: this.showFooter
                    }
                ];
            },
            bodyClasses () {
                return [
                    `${this.prefixCls}-body`,
                    {
                        [`${this.prefixCls}-body-with-search`]: this.filterable,
                        [`${this.prefixCls}-body-with-footer`]: this.showFooter
                    }
                ];
            },
            count () {
                const validKeysCount = this.validKeysCount;
                return (validKeysCount > 0 ? `${validKeysCount}/` : '') + `${this.data.length}`;
            },
            checkedAll () {
                return this.filterData.filter(data => !data.disabled).length === this.validKeysCount && this.validKeysCount !== 0;
            },
            checkedAllDisabled () {
                return this.filterData.filter(data => !data.disabled).length <= 0;
            },
            filterData () {
                // if (this.splitPage) {
                //     // let startIndex=(this.page-1)*this.splitPage;
                //     // let endIndex=(this.page)*this.splitPage;
                //     let sortArr=this.showItems.filter(item => this.filterMethod(item, this.query)).slice(startIndex,endIndex);
                //     return sortArr;
                // } else {
                return this.showItems.filter(item => this.filterMethod(item, this.query));
                // }
            }
        },
        methods: {
            lastPage(val){
                this.$emit('lastPage',val);
                //val==0 代表首页  0会转为false
                // if (val) {
                //     if (this.page>1) {
                //         this.page--;
                //     }
                // } else {
                //     this.page=1;
                // }
            },
            nextPage(val){
                this.$emit('nextPage',val);
                // const itemLength=Math.ceil(this.showItems.filter(item => this.filterMethod(item, this.query)).length/this.splitPage)-1;
                // if (val) {
                //     if (this.page<itemLength) {
                //         this.page++;
                //     }
                // } else {
                //     this.page=itemLength;
                // }
            },
            itemClasses (item) {
                return [
                    `${this.prefixCls}-content-item`,
                    {
                        [`${this.prefixCls}-content-item-disabled`]: item.disabled
                    }
                ];
            },
            showLabel (item) {
                return this.renderFormat(item);
            },
            isCheck (item) {
                return this.checkedKeys.some(key => key === item.key);
            },
            select (item) {
                if (item.disabled) return;
                const index = this.checkedKeys.indexOf(item.key);
                index > -1 ? this.checkedKeys.splice(index, 1) : this.checkedKeys.push(item.key);
                this.$parent.handleCheckedKeys();
            },
            updateFilteredData () {
                this.showItems = this.data;
            },
            toggleSelectAll (status) {
                const keys = status ?
                        this.filterData.filter(data => !data.disabled || this.checkedKeys.indexOf(data.key) > -1).map(data => data.key) :
                        this.filterData.filter(data => data.disabled && this.checkedKeys.indexOf(data.key) > -1).map(data => data.key);
                this.$emit('on-checked-keys-change', keys);
            },
            handleQueryClear () {
                this.query = '';
            },
            handleQueryChange (val) {
                this.query = val;
            }
        },
        created () {
            this.updateFilteredData();
        },
        mounted () {
            this.showFooter = this.$slots.default !== undefined;
        }
    };
</script>
<style scoped lang="less">
.ivu-transfer-split-page{
    @color:#dcdee2;
    @hoverColor:#2D8cF0;
    position: absolute;
    bottom: 0;
    padding-left: 2px;
    padding-right: 2px;
    font-size: 12px;
    width: 100%;
    display: flex;
    color:@color;
    justify-content: space-between;
    input{
        width: 30px;border:none;text-align: center;outline:none;color:black;
    }
    span{
        border-radius:2px;
        border: 1px @color solid;
        cursor: pointer;
    }
    span:hover{
        color:@hoverColor;
        border: 1px @hoverColor solid;
    }
    .right{
        margin-right: 5px;
    }
}
</style>
