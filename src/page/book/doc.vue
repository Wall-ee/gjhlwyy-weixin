<template>
  <div class="page">
    <app-header title="医生" ref="header">
      <i slot="back" class="back"></i>
    </app-header>
    <div class="main bg overflow-y-auto overflow-touch" ref="main">
      <div class="doc flex bg_white">
        <div class="ava flex0">
          <img class="radius50" :src="info.docAvatar" @error="__docError" alt="">
        </div>
        <div class="flex1 info lh1">
          <h3 class="color_333 fs16">
            {{info.docName}} <span class="fs14 color_333">{{info.docTitle}}</span>
          </h3>
          <div class="color_666">{{info.hosName}}</div>
          <p class="color_666">{{dept.length>0?dept[0].deptName:""}}</p>
        </div>
      </div>
      <div class="hos overflow-x-auto" v-if="hos.length>1">
        <ul class="flex bg_white">
          <router-link tag="li" :to="{path:'/book/doc/'+item.bookDocId}" replace v-for="(item,index) in hos"
                       :class="[id==item.bookDocId?'cover':'']" :key="index"
                       class="flex0 fs14 color_666 bg_white">
            {{item.bookHosName|hosName}}
          </router-link>
        </ul>
      </div>

      <div class="pblist">
        <ul>
          <li v-for="(o,i) in dept">
            <h3 class="flex color_999 fs14">
              <span class="flex1">{{o.deptName}}</span>
              <font class="flex0 iconfont">&#xe72b;</font>
            </h3>
            <ol>
              <li class="flex fs14" v-for="(oo,ii) in o.schemeList">
                <span class="flex0 mr5 color_333">{{oo.schemeDate.substr(0,10)}}</span>
                <span class="flex0 mr5 color_333">周{{oo.weekNo|week}}</span>
                <span class="flex1 color_333">{{oo.schemeAmpm|getOrderAmpm}}</span>
                <div class="flex0 num" :class="['num'+oo.schemeStats]">
                  <span v-if="oo.schemeStats==4" @click="num(oo)">预约 {{oo.bookFee}}元</span>
                  <span class="color_777" v-else>{{oo.schemeStats|schemeStatus}}</span>
                </div>
              </li>
            </ol>
          </li>
        </ul>
      </div>

      <div class="skill bg_white">
        <h3 class="fs14 color_999">医生擅长</h3>
        <div class="fs12 color_333">{{info.docDescription}}</div>
      </div>


      <!--<div class="dept bg_white">
        <ol v-if="dept.length>0" class="flex flex-center">
          <li class="flex0 fs14" :class="[index==currentDept?'cover':'']" v-for="(item,index) in dept" :key="index"
              @click="currentDept=index">
            {{item.deptName}}
          </li>
        </ol>
        <ol v-else class="flex flex-center flex-center-2 color_666 fs16">
          暂无排班信息
        </ol>
      </div>-->
      <!--<div class="list">
        <ul>
          <li class="flex flex-center bg_white" v-for="(item,index) in list" :key="index">
            <span class="flex0 mr10 color_333">{{item.schemeDate.substr(0,10)}}</span>
            <span class="flex0 mr10 color_333">周{{item.weekNo|week}}</span>
            <span class="flex1 color_333">{{item.schemeAmpm|getOrderAmpm}}</span>
            <div class="flex0 num" :class="['num'+item.schemeStats]">
              <span v-if="item.schemeStats==4" @click="num(item)">{{item.numUnlock}}/{{item.numCount}}</span>
              <span class="color_777" v-else>{{item.schemeStats|schemeStatus}}</span>
            </div>
          </li>
        </ul>
      </div>-->
    </div>
    <num-modal ref="num"></num-modal>
  </div>
</template>

<script>
  import AppHeader from "../../plugins/app-header";
  import {avaErrorMixin, isBindMixin, mainHeightMixin} from "../../lib/mixin";
  import weuijs from 'weui.js'
  import api from '../../lib/api'
  import {getOrderAmpm, schemeStatus, week} from "../../lib/filter";
  import NumModal from '../../plugins/book/num-modal'

  export default {
    data() {
      return {
        id: '',
        page: "",
        hos: [],
        info: {},
        dept: [],
        currentDept: 0
      };
    },
    computed: {
      list() {
        if (this.dept.length > 0) {
          return this.dept[this.currentDept].schemeList
        }
        return []
      }
    },
    filters: {
      week, getOrderAmpm, schemeStatus,
      hosName(str) {
        if (!str) {
          return ""
        }
        return str.replace("浙江大学医学院附属第二医院", "浙医二院");
      }
    },
    components: {AppHeader, NumModal},
    mixins: [mainHeightMixin, isBindMixin, avaErrorMixin],
    async created() {
      let {params, query} = this.$route;
      this.id = params.id;
      this.page = query.page || '';
      await this.getData();
      await this.getList();
    },
    mounted() {

    },
    beforeDestroy() {

    },
    watch: {
      async $route(newV, oldV) {
        if (newV !== oldV && oldV) {
          this.id = newV.params.id;
          await this.getData();
          await this.getList();
        }
      }
    },
    methods: {
      async num(data) {
        let ret = await this._isBind()
        if (ret) {
          this.$refs.num.show(data);
        } else {
          this.$router.push({
            path: "/bindRelativePhone",
            query: {backPath: this.$route.fullPath}
          });
        }
      },

      async getList() {
        let loading = weuijs.loading("加载中...");
        let ret = await api('nethos.sys.bookdoc.list', {bookDocId: this.id})
        loading.hide();
        if (ret.code == 0) {
          this.hos = ret.list;
        } else {
          //this.$refs.msg.show(ret.msg||"接口错误"+ret.code);
        }
      },
      async getData() {
        let loading = weuijs.loading("加载中...");
        let ret = await api('nethos.book.doc.scheme.list', {
          bookDocId: this.id
        })
        loading.hide();
        if (ret.code == 0) {
          this.info = ret.obj;
          this.dept = ret.obj.deptSchemeList;
        } else {
          //this.$refs.msg.show(.msg||"接口错误"+.code);
        }

      }
    }
  };
</script>

<style scoped lang="scss">
  @import "../../common/public";

  .pblist {
    margin-top: 10px;
    padding: 10px;
    background-color: white;
    h3 {
      padding: 5px 0;
    }
    ol > li {
      padding: 10px 0;
    }
  }

  .mr5 {
    margin-right: 5px;
  }

  .mr10 {
    margin-right: 10px;
  }

  .doc {
    padding: $commonSpace;
    $h: px2rem(160px, 1080);
    .ava {
      @include thumb($h, $h);
      margin-right: $commonSpace;
    }
    .info {
      div, p {
        margin-top: $commonSpace / 2;
      }
    }
  }

  .hos {
    ul {
      li {
        padding: 0 $commonSpace;
        &.cover {
          @include border(bottom, $mainColor, after, solid, 4px);
          color: $mainColor;
        }
      }
      @include h_lh(px2rem(130px, 1080));
    }
  }

  .dept {
    @include border();
    margin-top: $commonSpace;
    ol {
      height: px2rem(200px, 1080);
      padding: 0 $commonSpace;
      li {
        color: #666666;
        margin-right: $commonSpace;
        border: 1px solid $mainColor;
        padding: 2px 5px;
        border-radius: 10px;
        &.cover {
          color: $mainColor;
        }
      }
    }
  }

  /*.list {
    li {
      padding: 0 $commonSpace;
      height: 45px;
      .num.num4 {
        span {
          background-color: $mainColor;
          color: white;
          border-radius: 5px;
          padding: 3px 10px;
        }
      }
    }
  }*/

  .num {
    span {
      background-color: $bgColor;
      color: #666;
      border-radius: 5px;
      padding: 3px 10px;
    }
  }

  .num.num4 {
    span {
      background-color: $mainColor;
      color: white;
      border-radius: 5px;
      padding: 3px 10px;
    }
  }

  .skill {
    padding: 10px;
    margin-top: 10px;
  }
</style>
