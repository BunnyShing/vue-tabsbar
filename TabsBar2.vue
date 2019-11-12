<template>
  <el-tabs v-model="active" type="card" @tab-remove="removeTab">
    <el-tab-pane :closable="dontDelete && dontDelete.indexOf(getValueByKey(value,indexKey)) > -1 ? false : true"
                 v-for="(value,key,index) in tabsArr" :key="value.indexHash"
                 :label="getValueByKey(value,titleKey)"
                 :name="value.indexHash"
    ></el-tab-pane>
  </el-tabs>
</template>

<script>
  /*
  * 作者：JC_Ho
  * 邮箱：335836557@qq.com
  * 这是一个基于vue、vue-router、elementUI的多选项卡组件2.0版本
  * 请确保已经引入了vue-router和elementUI
  * 该组件是通过sessionStorage实现tab状态保持的
  * 注意：
  * 1、在开发过程中路由的新增、减少可能会导致tabs队列不正常，只要清除sessionStorage并刷新页面即可，正常上线后不会出现该问题。
  * 2、目前未测试过在动态路由的情况下是否会有BUG。
  * 调用示例：
  * <tabs-bar :dont-delete="['index']" :dont-listen="['login']" :multiple="['test']" index-key="name" title-key="meta.title"></tabs-bar>
  * */
    export default {
        props: {
            dontDelete: {//不能删除tab数组
                type: Array,
                default: []
            },
            dontListen: {//无需监听tab数组
                type: Array,
                default: []
            },
            multiple:{//可重复的tab数组
                type: Array,
                default: []
            },
            titleKey: {//tab的名称在router中的参数键名
                type: String,
                default: ''
            },
            indexKey: {//tab的索引在router中的参数键名
                type: String,
                default: ''
            },
        },
        name: 'TabsBar',
        data() {
            return {
                //当前选中的tab
                active: null,
                //tab队列，默认是展开全部不可删除的tab
                tabsArr: this.$router.options.routes.filter((routerItem, index, arr) => {
                    return this.dontDelete && this.dontDelete.indexOf(this.getValueByKey(routerItem, this.indexKey)) > -1 ? true : false
                }).map((routerItem) => {
                    let to = this.getInfoFromRoute(routerItem)
                    to.indexHash = this.getTabIndexHash(routerItem)
                    return to;
                })
            };
        },
        watch: {
            //监听当前选中的tab，并存到sessionStorage中
            active(newVal,oldVal) {
                sessionStorage.setItem('activeTab', newVal)
                //如果当前路由已经等于this.active了，那么就不再执行下去，否则会导致vue-router报错
                if( newVal === this.active){
                    return
                }
                this.$router.push(JSON.parse(newVal))
            },
            //监听tab队列，并存到sessionStorage中
            tabsArr(newVal) {
                sessionStorage.setItem('oldTabs', JSON.stringify(newVal))
            },
            //监听路由变化，并根据路由变化切换tab
            $route(to, from) {
                this.routerChange(to)
            }
        },
        mounted() {
            //启动时获取sessionStorage中保存的旧tabs队列，以实现浏览器刷新也不会重置tabs队列
            if (sessionStorage.getItem('oldTabs')) {
                this.tabsArr = JSON.parse(sessionStorage.getItem('oldTabs'))
            }
            //启动时获取sessionStorage中保存的已选中的选项卡，以实现浏览器刷新也能保持已选中
            if (sessionStorage.getItem('activeTab')) {
                this.active = sessionStorage.getItem('activeTab')
            }
            //启动时检查当前路由是否已选中，不是的话就改变tabs队列
            this.routerChange(this.$router.history.current)
        },
        methods: {
            //监听到路由变化，并实际操作tabs队列的函数
            routerChange(to) {
                //如果是未定义的路由，tabs队列不发生变化
                let pathArr = this.$router.options.routes.map((routerItem) => {return this.getValueByKey(routerItem,this.indexKey)})
                if(pathArr.indexOf(this.getValueByKey(to,this.indexKey)) === -1){return}

                //在[无需监听tab数组]中，tabs队列不发生变化
                if (this.dontListen.indexOf(this.getValueByKey(to,this.indexKey)) > -1) {return}

                let routerExist = false
                for (let i of this.tabsArr) {
                    //检查跳转的路由是否已存在于tabs队列。如果是，就不新增tabs，只激活选中的tabs
                    if(this.multiple.indexOf(this.getValueByKey(to, this.indexKey)) < 0){
                        if (this.getValueByKey(to, this.indexKey) === this.getValueByKey(i, this.indexKey)) {
                            routerExist = true
                            this.active = i.indexHash
                        }
                    }else {
                        //即使是可重复的路由，如果已经存在indexHash一样的，也不新增了，而且跳转到对应的tab
                        if(i.indexHash === this.getTabIndexHash(to)){
                            routerExist = true
                            this.active = i.indexHash
                        }
                    }
                }
                //如果路由不存在就新增一个tab，并让其选中
                if (!routerExist) {
                    let routerItem = this.getInfoFromRoute(to)
                    routerItem.indexHash = this.getTabIndexHash(to)
                    this.tabsArr.push(routerItem)
                    this.active = routerItem.indexHash
                }
            },
            //删除tab，删除后，激活前一个tab
            removeTab(indexHash) {
                let tabs = this.tabsArr;
                if (this.active === indexHash) {
                    tabs.forEach((tab, index) => {
                        if (tab.indexHash === indexHash) {
                            let nextTab = tabs[index + 1] || tabs[index - 1];
                            if (nextTab) {
                                this.active = nextTab.indexHash;
                            }
                        }
                    });
                }
                this.tabsArr = tabs.filter(tab => tab.indexHash !== indexHash);
            },
            //生成一个tab的唯一索引值
            getTabIndexHash(to){
                return JSON.stringify(this.getInfoFromRoute(to))
            },
            //根据key从对象中抽取出值
            getValueByKey(routerItem, key) {
                let indexKeyArr = key.split('.')
                let tempValue = routerItem
                for (let i of indexKeyArr) {
                    tempValue = tempValue[i]
                }
                return tempValue
            },
            //获取对象属性（不包含继承属性与__proto__）
            getObjWithOwnPrototype(obj){
                let resultObj = {}
                for (let i in obj){
                    if(obj.hasOwnProperty(i)){
                        resultObj[i] = typeof i === "object"?this.getObjWithOwnPrototype(i):obj[i]
                    }
                }
                return resultObj
            },
            //获取路由对象中有用且可存到sessionStorage中的数据
            getInfoFromRoute(to){
                return {
                    fullPath:to.fullPath,
                    hash:to.hash,
                    path:to.path,
                    params:to.params,
                    query:to.query,
                    name:to.name,
                    meta:to.meta,
                    alias:to.alias
                }
            }
        },
    }
</script>

<style scoped>

</style>
